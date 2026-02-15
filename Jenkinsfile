def SVN_BASE_URL = "https://svn.altoqi.com.br/svn/Eberick/"
def CHECKOUT_FOLDER_PREFIX = "FontesEberick"
def SCRIPTS_FOLDER = "Scripts"
def CMAKE_BINARY_DIR = "exeobj_cmake\\VC"
def OWL_VERSION = "6.42"

def SVN_URL = ""
def CHECKOUT_FOLDER = ""
def CHECKOUT_PATH = ""
def SCRIPTS_PATH = ""
def CMAKE_PROJECT_FOLDER = ""
def CMAKE_INSTALL_FOLDER = ""
def CMAKE_INSTALL_PREFIX = ""
def PYTHON_BUILD_COMMAND_FLAGS = ""

pipeline {
  agent any

  parameters {
    string(name: "REVISAO_SVN", defaultValue: "HEAD", description: "Revisao do SVN a ser compilada")
    string(name: "BRANCH_SVN", defaultValue: "trunk", description: "Branch do SVN a ser compilada")
    choice(name: "TIPO_BUILD", choices: ["RELEASE", "DEBUG"], description: "Tipo de construcao")
    choice(name: "TRADUCAO", choices: ["DESABILITADA", "PT-BR", "ES-MX", "EN-US"], description: "Suporte a traducao")
    choice(name: "PROTECAO", choices: ["OAUTH", "RMS", "DEMO", "SL"], description: "Tipo de protecao")
    booleanParam(name: "COTIRE", defaultValue: true, description: "Compilar com Unity Build?")
    booleanParam(name: "VLD", defaultValue: false, description: "Compilar com Visual Leak Detector?")
  }

  stages {

    stage('Configurar') {
      steps {

        // Popula dados do workspace com base nos parametros de SVN
        script {
          def rota = params.BRANCH_SVN == "trunk" ? "trunk" : "branches/${params.BRANCH_SVN}"
          SVN_URL = "${SVN_BASE_URL}${rota}"

          CHECKOUT_FOLDER = "${CHECKOUT_FOLDER_PREFIX}_${params.BRANCH_SVN}"
          CHECKOUT_PATH = "${env.WORKSPACE}\\${CHECKOUT_FOLDER}"
          SCRIPTS_PATH = "${CHECKOUT_PATH}\\${SCRIPTS_FOLDER}"
        }

        // Gera o comando de build com base nas opcoes selecionadas
        script {
          if (params.TIPO_BUILD == "DEBUG") {
            PYTHON_BUILD_COMMAND_FLAGS += "-B=deb"
          } else {
            PYTHON_BUILD_COMMAND_FLAGS += "-B=rel"
          }

          PYTHON_BUILD_COMMAND_FLAGS += " -V=22 -TS=143"

          PYTHON_BUILD_COMMAND_FLAGS += " "
          if (params.PROTECAO == "SL") {
            PYTHON_BUILD_COMMAND_FLAGS += "-SL"
          } else if (params.PROTECAO == "DEMO") {
            PYTHON_BUILD_COMMAND_FLAGS += "-D"
          } else if (params.PROTECAO == "RMS") {
            PYTHON_BUILD_COMMAND_FLAGS += "-R"
          } else {
            PYTHON_BUILD_COMMAND_FLAGS += "-O"
          }

          if (params.TRADUCAO != "DESABILITADA") {
            PYTHON_BUILD_COMMAND_FLAGS += " -T -Lang="
            if (params.TRADUCAO == "EN-US") {
              PYTHON_BUILD_COMMAND_FLAGS += "en-US"
            } else if (params.TRADUCAO == "ES-MX") {
              PYTHON_BUILD_COMMAND_FLAGS += "es-MX"
            } else {
              PYTHON_BUILD_COMMAND_FLAGS += "pt-BR"
            }
          }

          if (params.COTIRE) {
            PYTHON_BUILD_COMMAND_FLAGS += " -C"
          }

          if (params.VLD) {
            PYTHON_BUILD_COMMAND_FLAGS += " -L"
          }

          PYTHON_BUILD_COMMAND_FLAGS += " -P2"
        }

        // Monta nome da pasta a ser gerada pelo comando de build
        script {
          if (params.TIPO_BUILD == "DEBUG") {
            CMAKE_PROJECT_FOLDER += "deb"
          } else {
            CMAKE_PROJECT_FOLDER += "rel"
          }

          CMAKE_PROJECT_FOLDER += "_vs22_msvc143"

          if (params.PROTECAO == "SL") {
            CMAKE_PROJECT_FOLDER += "_sl"
          } else if (params.PROTECAO == "DEMO") {
            CMAKE_PROJECT_FOLDER += "_demo"
          } else if (params.PROTECAO == "RMS") {
            CMAKE_PROJECT_FOLDER += "_rms"
          } else {
            CMAKE_PROJECT_FOLDER += "_oauth"
          }

          if (params.TRADUCAO != "DESABILITADA") {
            CMAKE_PROJECT_FOLDER += "_tr"
          }

          if (params.COTIRE) {
            CMAKE_PROJECT_FOLDER += "_cotire"
          }

          if (params.VLD) {
            CMAKE_PROJECT_FOLDER += "_vld"
          }
        }

        // Monta nome da pasta do comando de instalacao do CMake
        script {
          if (params.TIPO_BUILD == "DEBUG") {
            CMAKE_INSTALL_FOLDER += "Db"
          } else {
            CMAKE_INSTALL_FOLDER += "Rel"
          }

          CMAKE_INSTALL_FOLDER += "_OWL${OWL_VERSION}"

          if (params.VLD) {
            CMAKE_INSTALL_FOLDER += "_VLD"
          }

          if (params.PROTECAO != "DEMO") {
            if (params.PROTECAO == "SL") {
              CMAKE_INSTALL_FOLDER += "_SL"
            } else if (params.PROTECAO == "RMS") {
              CMAKE_INSTALL_FOLDER += "_RMS"
            } else {
              CMAKE_INSTALL_FOLDER += "_OAUTH"
            }
          }

          CMAKE_INSTALL_FOLDER += "_x64"

          if (params.PROTECAO == "DEMO") {
            CMAKE_INSTALL_FOLDER += "_Demo"
          }

          if (params.COTIRE) {
            CMAKE_INSTALL_FOLDER += "_COTIRE"
          }

          if (params.TRADUCAO != "DESABILITADA") {
            CMAKE_INSTALL_FOLDER += "_TRA"
          }

          CMAKE_INSTALL_PREFIX = "${CHECKOUT_PATH}\\${CMAKE_BINARY_DIR}\\${CMAKE_INSTALL_FOLDER}"
        }

      }
    }

    stage("Espelhar") {
      steps {
        script {
          bat "svn checkout -r ${params.REVISAO_SVN} ${SVN_URL} ${CHECKOUT_FOLDER}"
        }
      }
    }

    // stage("Analisar") {
    //   steps {
    //     echo CHECKOUT_PATH
    //   }
    // }

    stage("Construir") {
      steps {
        dir(SCRIPTS_PATH) {
          bat "python build.py ${PYTHON_BUILD_COMMAND_FLAGS}"
        }
      }
    }

    // stage("Envelopar") {
    //   steps {
    //     bat "${SCRIPTS_PATH}\\ProtegeEberick.bat"
    //   }
    // }

    // stage("Distribuir") {
    //   steps {
    //     echo "Distribuindo"
    //   }
    // }

    // stage("Testar") {
    //   steps {
    //     echo "Testando"
    //   }
    // }

  }
}
