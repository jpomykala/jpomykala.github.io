---
layout: post
title: "How I created native CLI in Java 11" 
categories: [software-development, internationalization, i18n, react, software, startup]
---


https://micronaut.io/launch/
https://github.com/simplelocalize/simplelocalize-cli

# Delombok

# Adjust reflections


#Github Action Pipeline

{% highlight yaml %}
name: Build executables

jobs:
  build-jar:
      name: "Build JAR"
      runs-on: ubuntu-latest
      steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-java@v1
        with:
          java-version: '11'
      - name: "Cache Maven packages"
        uses: actions/cache@v2
        with:
          path: ~/.m2
          key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}
          restore-keys: ${{ runner.os }}-m2
      - name: "Build with Maven"
        run: mvn -B clean install --file pom.xml
      - name: 'Get Version Number'
        run: echo "::set-env name=VERSION::$(mvn help:evaluate -Dexpression=project.version -q -DforceStdout)"
      - name: 'Upload artifact'
        uses: actions/upload-artifact@v2
        with:
          name: 'simplelocalize-cli-${{env.VERSION}}.jar'
          path: 'target/simplelocalize-cli-${{env.VERSION}}.jar'
  build-windows:
    needs: [build-jar]
    name: "Build Windows executable"
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v2
      - name: "Download GraalVM"
        run: |
          Invoke-RestMethod -Uri https://github.com/graalvm/graalvm-ce-builds/releases/download/vm-20.1.0/graalvm-ce-java11-windows-amd64-20.1.0.zip -OutFile 'graal.zip'
      - name: "Install GraalVM"
        run: |
          Expand-Archive -path 'graal.zip' -destinationpath '.'
      - name: "Install Native Image"
        run: |
          graalvm-ce-java11-20.1.0\bin\gu.cmd install native-image
      - name: "Set up Visual C Build Tools Workload for Visual Studio 2017 Build Tools"
        run: |
          choco install visualstudio2017-workload-vctools
      - name: 'Get Version Number'
        run: |
          echo "::set-env name=VERSION::$(mvn help:evaluate -Dexpression=project.version -q -DforceStdout)"
        shell: bash
      - name: 'Get JAR Artifact'
        uses: actions/download-artifact@v2
        with:
          name: 'simplelocalize-cli-${{env.VERSION}}.jar'
      - name: "Build Native Image"
        shell: cmd
        env:
          JAVA_HOME: ./graalvm-ce-java11-20.1.0
        run: |
          call "C:\Program Files (x86)\Microsoft Visual Studio\2017\BuildTools\VC\Auxiliary\Build\vcvars64.bat"
          ./graalvm-ce-java11-20.1.0/bin/native-image --no-server --report-unsupported-elements-at-runtime -cp simplelocalize-cli-${{env.VERSION}}.jar -H:Name="simplelocalize-cli" io.simplelocalize.cli.SimplelocalizeCliCommand
      - name: 'Upload artifact'
        uses: actions/upload-artifact@v2
        with:
          name: simplelocalize-cli-windows
          path: 'simplelocalize-cli.exe'

  build-unix:
    needs: [build-jar]
    name: "Build Unix executable"
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest]
        include:
          - os: 'ubuntu-latest'
            label: 'linux'
          - os: 'macos-latest'
            label: 'mac'
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v2
      - name: 'Get Version Number'
        run: |
          echo "::set-env name=VERSION::$(mvn help:evaluate -Dexpression=project.version -q -DforceStdout)"
      - name: 'Get JAR Artifact'
        uses: actions/download-artifact@v2
        with:
          name: 'simplelocalize-cli-${{env.VERSION}}.jar'
      - name: "Setup GraalVM"
        uses: DeLaGuardo/setup-graalvm@3
        with:
          graalvm-version: '20.1.0.java11'
      - name: "Install Native Image"
        run: gu install native-image
      - name: "Build Native Image"
        run: native-image --no-server --report-unsupported-elements-at-runtime -cp simplelocalize-cli-*.jar -H:Name="simplelocalize-cli" io.simplelocalize.cli.SimplelocalizeCliCommand
      - name: "Upload artifact"
        uses: actions/upload-artifact@v2
        with:
          name: simplelocalize-cli-${{ matrix.os }}
          path: simplelocalize-cli
{% endhighlight %}


### Sources
https://blogs.oracle.com/developers/building-cross-platform-native-images-with-graalvm
Todd Sharp

https://youtu.be/Xdcg4Drg1hc 
Szymon Stepniak
