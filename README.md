# Godot - Compilar Módulos para Android
Abaixo faço um passo-a-passo compilando um módulo do Firebase utilizando o Ubuntu 18.04 LTS

## Nesse tutorial foi utilizado
- Godot 3.1beta11
- java version "1.8.0_202"
- Java(TM) SE Runtime Environment (build 1.8.0_202-b08)
- Java HotSpot(TM) 64-Bit Server VM (build 25.202-b08, mixed mode)
- scons script: v3.0.1.74
- scons engine: v3.0.1.74
- Gradle 3.4.1
- Ubuntu 18.04 LTS
- git version 2.17.1
- Veja todos os requisitos necessários na documentação da Godot ( https://goo.gl/xNn68f )

### - Criar conta no Firebase
Acesse e crie uma conta: ```http://firebase.google.com```

- Clicar na opção ```Android``` e ```registrar o APP```
- Fazer o download do "```google-services.json```"
- Acesse: ```Configurações no Firebase -> DEVELOP -> Authentication -> Métodos de login```
    - Habilitar autenticação com e-mail e senha
- Acesse: ```Configurações em Firebase -> DEVELOP -> Database```
	- Criar novo banco de dados Cloud Firestore

### - Dependências
Faça os clones utilizando os comandos do github

- Baixar (Godot 3.1beta): git clone https://github.com/godotengine/godot
- Baixar: git clone https://github.com/FrogSquare/GodotSQL
- Baixar: git clone https://github.com/FrogSquare/GodotFireBase


### - Configurações Antes da Compilação
Copie seu arquivo ```google-services.json``` para ```[GODOT-ROOT]/platform/android/java/```
e edite o arquivo ```modules/FireBase/config.py```, substituindo ```com.your.appid``` pelo package do seu projeto.

```java
    env.android_add_default_config("applicationId 'com.your.appid'")
```

### - Preparar Ambiente Ubuntu (para compilação)
- Instalar Android Studio ( https://goo.gl/xrJwYe )
    - Utilize o SDK Manager para instalar os requisitos que constam na documentação da Godot:
        - Android SDK version 23.0.3 [Note: Please install all tools and extras of the SDK Manager]
        - Android build tools version 19.1
        - Android NDK r13 or later

- Instalar bilbiotecas 64bits:
	- Ubuntu: ```sudo apt-get install libc6:i386 libncurses5:i386 libstdc++6:i386 lib32z1 libbz2-1.0:i386```

- Verificar se o Python está instalado (testei com a versão: 2.7.15rc1)
	- ```python --version```

- Instalar scons (testei com a versão 3.0.1):
	- ```scons --version```
	    - Se não estiver instalado rodar: ```sudo apt install scons```

- Verificar se java está instalado (testei com a versão: 1.8.0_201)
	- ```java -version```
	    - Se não estiver instalado, veja como instalar: ( https://goo.gl/cjgBi2 )

- Configurar variáveis do ambiente no arquivo: 
	- Fazendo dessa forma como explico abaixo, se você fechar o terminal, não tem problema, não precisa rodar isso tudo de novo.
	- Execute o comando abaixo para editar o arquivo ```~/.bashrc```
	- ```nano ~/.bashrc```
	    - Colocar os caminhos abaixo, no fim do arquivo

    ```bash
        export ANDROID_HOME="/home/$USER/Android/Sdk"
        export ANDROID_NDK_ROOT="/home/$USER/Android/Sdk/ndk-bundle"

        PATH=${PATH}:${ANDROID_HOME}
        PATH=${PATH}:${ANDROID_HOME}/tools
        PATH=${PATH}:${ANDROID_HOME}/platform-tools
        export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_202
    ```

	- [Onde tem JAVA_HOME acima], precisa só conferir se o caminho está certo para seu java, e colocar o certo no seu pc
	- Após editar o arquivo, execute o comando abaixo para recarregar o arquivo e não precisar reiniciar o pc
	- ```source ~/.bashrc```

	- Agora apenas para testar, digite no terminal
	- ```echo $ANDROID_HOME``` (veja se apareceu o caminho)
	- ```cd $ANDROID_HOME``` (veja se entrou no caminho)

	- ```echo $ANDROID_NDK_ROOT``` (veja se apareceu o caminho)
	- ```cd $ANDROID_NDK_ROOT``` (veja se entrou no caminho)

	- ```echo $JAVA_HOME``` (veja se apareceu o caminho)
	- ```cd $JAVA_HOME``` (veja se entrou no caminho)


### - compilar para RELEASE primeiro
- Acesse o diretório principal da Godot (source) 
    * Nesse diretório vc vai trabalhar a maior parte do tempo abaixo.\
    * A primeira vez vai demorar bastante! Pode ir fazer uns 5 cafés!!! kkkk

Execute:
- ```scons platform=android target=release```

- Depois de compilar, acesse: "```cd platform/android/java```", e execute:

- ```nano build.gradle```
- localize onde tem a chave "Android" e acrescente a informação "dexOptions" abaixo:
```java
android {
    dexOptions {
        jumboMode = true
    }
    ... #mantenha os outros códigos que estiverem no arquivo build.gradle
}
```

- Depois execute esse comando:
- ```./gradlew build```


### - Compilar para debug:
- Retorne ao diretório principal (```cd ../../../```) da godot (source) e execute agora em modo debug:
- ```scons platform=android target=release_debug```
*** Talvez essa demore mais que a primeira vez!

- Acesse: ```cd platform/android/java```
- Abra o arquivo build.gradle novamente
- ```nano build.gradle```

- Localize onde tem a chave "Android" e acrescente a informação dexOptions abaixo:
```java
android {
    dexOptions {
        jumboMode = true
    }
    ... #mantenha os outros códigos que estiverem no arquivo build.gradle
}
```

- Salve o arquivo, e execute o comando abaixo:
- ```./gradlew build```


### - Na Godot
Se tudo ocorreu bem nas compilações, você pode abrir seu jogo, e no exports do Android, em custom package Debug e Release
coloca os arquivos que estão na pasta BIN da godot (source)\

No seu arquivo ```project.godot``` acrescente essa parte no final dele:

```text
[android]
modules="org/godotengine/godot/FireBase,org/godotengine/godot/SQLBridge"
```

Tente rodar no celular e por enquanto parece que está tudo ok se abrir!
Se o app abrir e fechar falando que travou, então seu módulo ficou com problema, precisa refazer os passos acima, observando se não deu erro em nenhuma parte da compilação, a maioria dos problemas é por questões de versão, ou o java está com versão errada, ou o scons ou gradlew.

Agora o teste de minerva!!!
- Crie um script em seu node principal com o código abaixo:
** Não esqueça de criar o arquivo ```godot-firebase-config.json``` no diretório do seu jogo com suas configurações que vc quer.
** Não esqueça de habilitar na exportação para Android, pra godot colocar os arquivos ```*.json``` junto com seu pacote!


```python
var firebase

func _ready():
	if OS.get_name() == "Android":
		firebase = Engine.get_singleton("FireBase")
		firebase.initWithFile("res://godot-firebase-config.json", get_instance_id())

func _receive_message(tag, from, key, data):
	if tag == "FireBase":
		print("From: ", from, " Key: ", key, " Data: ", data)"

```

- com o comando abaixo, deixe rodando em um terminal enquanto vc abre o jogo no celular,
nele você vai conseguir ver caso dê erro na hora de carregar o Firebase, ou autenticar etc.

```bash
adb -d logcat godot:V FireBase:V DEBUG:V AndroidRuntime:V ValidateServiceOp:V *:S
```