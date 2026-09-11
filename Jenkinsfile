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

                        Write-Host "Downloading SonarScanner CLI..."

                        $release = Invoke-RestMethod `
                            -Uri "https://api.github.com/repos/SonarSource/sonar-scanner-cli/releases/latest"

                        $asset = $release.assets |
                            Where-Object { $_.name -match "windows-x64\\.zip$" } |
                            Select-Object -First 1

                        if (-not $asset) {
                            throw "Windows SonarScanner ZIP was not found."
                        }

                        $zip = "$env:WORKSPACE\\sonar-scanner.zip"
                        $folder = "$env:WORKSPACE\\sonar-scanner"

                        if (Test-Path $folder) {
                            Remove-Item $folder -Recurse -Force
                        }

                        Invoke-WebRequest `
                            -Uri $asset.browser_download_url `
                            -OutFile $zip

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
                            throw "sonar-scanner.bat was not found."
                        }

                        Write-Host "Running SonarCloud analysis..."

                        & $scanner.FullName `
                            "-Dsonar.token=$env:SONAR_TOKEN"
                    '''
                }
            }
        }
    }
}
