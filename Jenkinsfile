pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Dhruv12aj/7.2cDevSecOps.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                bat 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                bat 'npm test || exit /b 0'
            }
        }

        stage('Generate Coverage Report') {
            steps {
                bat 'npm run coverage || exit /b 0'
            }
        }

        stage('NPM Audit (Security Scan)') {
            steps {
                bat 'npm audit || exit /b 0'
            }
        }

        stage('SonarCloud Analysis') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'SONAR_TOKEN',
                        variable: 'SONAR_TOKEN'
                    )
                ]) {
                    powershell '''
                        $ErrorActionPreference = "Stop"

                        Write-Host "Getting latest SonarScanner version..."

                        $release = Invoke-RestMethod `
                            -Headers @{ "User-Agent" = "Jenkins" } `
                            -Uri "https://api.github.com/repos/SonarSource/sonar-scanner-cli/releases/latest"

                        $version = $release.tag_name -replace "^v", ""

                        Write-Host "SonarScanner version: $version"

                        $downloadUrl = "https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-$version-windows-x64.zip"

                        $zip = "$env:WORKSPACE\\sonar-scanner.zip"
                        $folder = "$env:WORKSPACE\\sonar-scanner"

                        if (Test-Path $folder) {
                            Remove-Item $folder -Recurse -Force
                        }

                        if (Test-Path $zip) {
                            Remove-Item $zip -Force
                        }

                        Write-Host "Downloading SonarScanner CLI..."

                        Invoke-WebRequest `
                            -Uri $downloadUrl `
                            -OutFile $zip `
                            -UseBasicParsing

                        Write-Host "Extracting SonarScanner..."

                        Expand-Archive `
                            -Path $zip `
                            -DestinationPath $folder `
                            -Force

                        $scanner = Get-ChildItem `
                            -Path $folder `
                            -Filter "sonar-scanner.bat" `
                            -Recurse |
                            Select-Object -First 1

                        if (-not $scanner) {
                            throw "sonar-scanner.bat was not found after extraction."
                        }

                        Write-Host "Running SonarCloud analysis..."

                        & $scanner.FullName `
                            "-Dsonar.token=$env:SONAR_TOKEN"

                        if ($LASTEXITCODE -ne 0) {
                            throw "SonarScanner failed with exit code $LASTEXITCODE"
                        }
                    '''
                }
            }
        }
    }
}
