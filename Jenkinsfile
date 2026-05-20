pipeline {

    agent any

    tools {
        jdk 'JDK21'
        maven 'maven'
    }

    environment {

        JAVA_PROJECT = "Java/JavaFullstackEcommerce"

        CPP_PROJECT = "Cpp"
        PYTHON_EXE = 'C:\\Program Files\\Python313\\python.exe'
        PYTHON_PARSER = "parser/parser.py"

        SONAR_TOKEN = credentials('sonar-token')
        CMAKE_EXE = 'C:\\Program Files\\CMake\\bin\\cmake.exe'

        CTEST_EXE = 'C:\\Program Files\\CMake\\bin\\ctest.exe'

        GCOVR_EXE = 'C:\\Program Files\\Python313\\Scripts\\gcovr.exe'
    }

    stages {

        // ============================================
        // SCM CHECKOUT INFO
        // ============================================
        stage('Repository Ready') {

            steps {

                echo "Repository already checked out by Jenkins SCM."

                bat """
                echo Current Workspace:
                cd

                echo Listing Root Files:
                dir
                """
            }
        }

        // ============================================
        // VERIFY STRUCTURE
        // ============================================
        stage('Verify Repository Structure') {

            steps {

                bat """
                echo VERIFYING JAVA STRUCTURE

                dir Java

                dir Java\\JavaFullstackEcommerce

                echo VERIFYING CPP STRUCTURE

                dir Cpp

                echo VERIFYING PARSER

                dir parser
                """
            }
        }

        // ============================================
        // JAVA BUILD + TEST + COVERAGE
        // ============================================
        stage('Java Build & Coverage') {

            steps {

                dir("${JAVA_PROJECT}") {

                    bat """
                    echo CURRENT DIRECTORY
                    cd

                    echo VERIFY POM
                    dir pom.xml

                    mvn clean verify
                    """
                }
            }
        }

        // ============================================
        // VERIFY JACOCO OUTPUT
        // ============================================
        stage('Verify Java Coverage Artifacts') {

            steps {

                bat """
                echo VERIFYING JACOCO FILES

                dir Java\\JavaFullstackEcommerce\\target\\site\\jacoco

                dir Java\\JavaFullstackEcommerce\\target\\surefire-reports
                """
            }
        }

        // ============================================
        // JAVA SONARCLOUD
        // ============================================
        stage('Java SonarCloud Analysis') {

            steps {

                dir("${JAVA_PROJECT}") {

                    withSonarQubeEnv('SonarCloud') {

                        bat """
                        mvn sonar:sonar ^
                        -Dsonar.projectKey=bmc-java ^
                        -Dsonar.organization=prathamesh-jawahire ^
                        -Dsonar.host.url=https://sonarcloud.io ^
                        -Dsonar.token=%SONAR_TOKEN% ^
                        -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
                        """
                    }
                }
            }
        }

        // ============================================
        // CPP BUILD + TEST + COVERAGE
        // ==========================================

    environment {

        CMAKE_EXE = 'C:\\Program Files\\CMake\\bin\\cmake.exe'

        CTEST_EXE = 'C:\\msys64\\mingw64\\bin\\ctest.exe'

        GCOVR_EXE = 'C:\\Users\\Prathemesh\\AppData\\Roaming\\Python\\Python313\\Scripts\\gcovr.exe'
    }
    stage('CPP Build & Coverage') {

    steps {

        dir("${CPP_PROJECT}") {

            bat """

            echo ======================================
            echo CPP BUILD START
            echo ======================================

            if exist build rmdir /s /q build

            mkdir build

            cd build

            echo ======================================
            echo USING MINGW GCC
            echo ======================================

            "C:\\Program Files\\CMake\\bin\\cmake.exe" ^
            -G "MinGW Makefiles" ^
            -DCMAKE_C_COMPILER=C:/msys64/mingw64/bin/gcc.exe ^
            -DCMAKE_CXX_COMPILER=C:/msys64/mingw64/bin/g++.exe ^
            ..

            echo ======================================
            echo BUILDING PROJECT
            echo ======================================

            "C:\\Program Files\\CMake\\bin\\cmake.exe" --build .

            echo ======================================
            echo RUNNING TESTS
            echo ======================================

            "C:\\Program Files\\CMake\\bin\\ctest.exe" --output-on-failure

            echo ======================================
            echo SEARCHING COVERAGE FILES
            echo ======================================

            dir /s *.gcda
            dir /s *.gcno

            echo ======================================
            echo GENERATING GCOVR REPORTS
            echo ======================================

            "C:\\Program Files\\Python313\\python.exe" -m gcovr ^
            -r .. ^
            --xml ^
            -o coverage.xml

            "C:\\Program Files\\Python313\\python.exe" -m gcovr ^
            -r .. ^
            --json ^
            -o coverage.json

            echo ======================================
            echo VERIFY GENERATED FILES
            echo ======================================

            dir

            """
        }
    }
}
        // ============================================
        // VERIFY CPP ARTIFACTS
        // ============================================
        stage('Verify CPP Coverage Artifacts') {

            steps {

                bat """
                echo VERIFYING CPP COVERAGE

                dir Cpp\\build
                """
            }
        }

        // ============================================
        // GENERATE UNIFIED JSON
        // ============================================        
        stage('Generate Unified JSON') {

    steps {

        bat """
        "%PYTHON_EXE%" parser/parser.py ^
        --jacoco_xml Java/JavaFullstackEcommerce/target/site/jacoco/jacoco.xml ^
        --surefire_dir Java/JavaFullstackEcommerce/target/surefire-reports ^
        --gcovr_xml Cpp/build/coverage.xml ^
        --gcovr_json Cpp/build/coverage.json ^
        --output unified_report.json
        """
    }
}

        // ============================================
        // VERIFY JSON OUTPUT
        // ============================================
        stage('Verify Unified JSON') {

            steps {

                bat """
                echo VERIFYING FINAL JSON

                dir unified_report.json

                type unified_report.json
                """
            }
        }

        // ============================================
        // ARCHIVE REPORTS
        // ============================================
        stage('Archive Reports') {

            steps {

                archiveArtifacts artifacts: '''
                unified_report.json,
                Java/JavaFullstackEcommerce/target/site/jacoco/**,
                Java/JavaFullstackEcommerce/target/surefire-reports/**,
                Cpp/build/coverage.xml,
                Cpp/build/coverage.json
                ''',
                fingerprint: true
            }
        }
    }

    post {

        always {

            echo "Pipeline Finished."
        }

        success {

            echo "Build Success."
        }

        failure {

            echo "Build Failed."
        }
    }
}