
# Github Actions for Prisma Cloud IaC Full Scan

This is a Github Actions workflow to perform Prisma Cloud Scans on:
- Code to build Docker images
- Terraform code
- Kubernetes manifests

The idea is to bring all the components of Prisma Cloud Scan together (Twistcli image scan, Twistcli sandbox scan and Bridgecrew scan) to scan differnt pieces of IaC in the same CICD run.

## Workflow Overview
The workflow consists of multipls jobs, each with its steps:

1. **dockerImages**: This job generates a matrix of all Docker image directories in the `docker-images/` directory. The matrix is passed to the next job to scan Docker images.
2. **docker-image-scan**: This job builds a Docker image from the specified directory and scans it using Prisma Cloud. If the image is not whitelisted, the twistcli sandbox is used to scan the image. If the image is whitelisted, sandbox analysis is skipped.
3. **terraformCode**: This job generates a matrix of all Terraform code directories in the `terraform/` directory. The matrix is passed to the next job to scan Terraform code.
4. **terraform-scan**: This job runs Bridgecrew to scan the Terraform code using Prisma Cloud. The scan results are saved in SARIF and console formats.
5. **kubernetesManifests**: This job generates a matrix of all Kubernetes code directories in the `kubernetes/` directory. The matrix is passed to the next job to scan Kubernetes manifests.
6. **kubernetes-scan**: This job runs Bridgecrew to scan the Kubernetes manifests using Prisma Cloud. The scan results are saved in SARIF and console formats.

## Required Workflow Secrets
Below are the secrets that are passed as variables to this Github Action:
- **DOCKERHUB_TOKEN:** Dockerhub personal access token
- **PCC_CONSOLE_URL:** Prisma Cloud Compute URL
- **PCC_ACCESS_KEY_ID:** Prisma Cloud Access Key ID
- **PCC_SECRET_ACCESS_KEY:** Prisma Cloud Secret Access Key
- **PC_API_KEY:** Prsima Cloud Access Key ID and Secret Key in the format `<PCC_SECRET_ACCESS_KEY>::<PC_API_KEY>`


## Workflow Details
### Prisma Cloud IaC Full Scan
The `docker-image-scan` job has the following steps:

1. **Checkout**: This step checks out the code from the repository.
2. **Docker Build**: This step builds a Docker image from the specified directory using the `Dockerfile` in that directory. The image is tagged with the latest `github.run_number` and the name of the directory. The resulting image is pushed to the local Docker registry.

    ```
    - name: Docker Build
      run: |
        docker build -t ultimate-test-drive/${{ matrix.path }}:latest -t ultimate-test-drive/${{ matrix.path }}:${{ env.SHA }} -f ${{ github.workspace }}/docker-images/${{ matrix.path }}/Dockerfile ${{ github.workspace }}/docker-images/${{ matrix.path }}
      shell: bash {0}  
    ```

3. **Prisma Cloud Image Scan**: This step uses the `PaloAltoNetworks/prisma-cloud-scan` action to scan the Docker image using Prisma Cloud. 

    ```
    - name: Prisma Cloud image scan
      id: scan
      uses: PaloAltoNetworks/prisma-cloud-scan@v1
      with:
        pcc_console_url: ${{ secrets.PCC_CONSOLE_URL }}
        pcc_user: ${{ secrets.PCC_ACCESS_KEY_ID }}
        pcc_pass: ${{ secrets.PCC_SECRET_ACCESS_KEY }}
        image_name: ultimate-test-drive/${{ matrix.path }}:latest
    ```

    The action requires the following inputs:
    - `pcc_console_url`: The URL of the Prisma Cloud Compute 
    - `pcc_user`: The access key ID for Prisma Cloud.
    - `pcc_pass`: The secret access key for Prisma Cloud.
    - `image_name`: The name of the Docker image to scan.

4. **Run twistcli sandbox**: This step runs the twistcli sandbox to perform a sandbox analysis of the Docker image. If the image is whitelisted, the sandbox analysis is skipped. 

    ```
    # Runs twistcli sandbox
    - name: Run twistcli sandbox
      run: |
        if [ ${{ matrix.path }} = "ubuntu" ]; then
          curl -s https://utd-packages.s3.amazonaws.com/twistcli --output twistcli
          chmod +x twistcli
          sudo ./twistcli sandbox --address ${{ secrets.PCC_CONSOLE_URL }} --user ${{ secrets.PCC_ACCESS_KEY_ID }} --password ${{ secrets.PCC_SECRET_ACCESS_KEY }} --analysis-duration 2m ultimate-test-drive/${{ matrix.path }}:latest
        else
          echo "Skipping Sandbox analysis for Whitelisted image: ${{ matrix.path }}"
        fi
    ```

### Terraform Scans
The `terraform-scan` job has the following steps:

1. **Checkout**: This step checks out the code from the repository.
2. **Run Bridgecrew**: This step runs Bridgecrew to scan the Terraform code using Prisma Cloud. 

    ```
    - name: Run Bridgecrew
      id: Bridgecrew
      uses: bridgecrewio/bridgecrew-action@master
      env:
        PRISMA_API_URL: https://api.eu.prismacloud.io    
      with:
        api-key: ${{ secrets.BC_API_KEY }}
        directory: "${{ github.workspace }}/terraform/${{ matrix.path }}"
        soft_fail: false
        framework: terraform
        output_format: cli,sarif
        output_file_path: console,results.sarif
        quiet: true
        download_external_modules: true
        log_level: WARNING
        check: CRITICAL,HIGH  
    ```

    The action requires the following inputs:
    - `api-key`: The API key for Bridgecrew.
    - `directory`: The path to the Terraform code directory to scan.
    - `framework`: The name of the framework to use for scanning (in this case, `terraform`).
    - `output_format`: The format(s) in which to output the scan results (in this case, `cli,sarif`).
    - `output_file_path`: The path(s) to save the scan results 
    - `quiet`: Display only failed checks
    - `download_external_modules`: download external terraform modules from public git repositories and terraform registry
    - `log_level`: 	set log level
    - `check`: Filter scan to run only on a specific check identifier, You can specify multiple checks separated by comma delimiter

### Kubernetes Scans
1. **Checkout**: This step checks out the source code of the pull request.
2. **Run Kubernetes Scan**: This step runs the Bridgecrew Github Action that performs a Kubernetes scan on the Kubernetes manifests in the repository using Prisma Cloud. 

    ```
    - name: Run Bridgecrew
      id: Bridgecrew
      uses: bridgecrewio/bridgecrew-action@master
      env:
        PRISMA_API_URL: https://api.eu.prismacloud.io    
      with:
        api-key: ${{ secrets.BC_API_KEY }}
        directory: "${{ github.workspace }}/kubernetes/${{ matrix.path }}"
        soft_fail: false
        framework: kubernetes
        output_format: cli,sarif
        output_file_path: console,results.sarif
        quiet: true
        download_external_modules: true
        log_level: WARNING
        skip_check: LOW,MEDIUM
        check: CKV_K8S*
    ```

    The action is run with the following parameters:
    - `api-key`: The Prisma Cloud API key stored as a secret in the repository.
    - `directory`: Root directory to scan.
    - `soft_fail`: Runs checks without failing build. Defaults to `false`.
    - `framework`: The infrastructure as code framework to use for the scan. In this case, it is set to `kubernetes`.
    - `output_format`: The format for the scan results. In this case, it is set to `cli,sarif`.
    - `output_file_path`: The path to the output file. In this case, it is set to `console,results.sarif`.
    - `quiet`: If set to `true`, the action will not output passed checks and will only print failed checks. Defaults to `true`.
    - `log_level`: The log level for the action. In this case, it is set to `WARNING`.
    - `skip_check`: Filter scan to run on all check but a specific check identifier(blacklist), You can specify multiple checks separated by comma delimiter, clashes with check. In this case, it is set to `LOW`.
    - `check`: Filter scan to run only on a specific check identifier, You can specify multiple checks separated by comma delimiter. In this case, it is set to `CKV_K8S*`.

## References
- https://github.com/bridgecrewio/bridgecrew-action    
- https://github.com/PaloAltoNetworks/prisma-cloud-scan
- https://github.com/docker/login-action
- https://github.com/docker/build-push-action
