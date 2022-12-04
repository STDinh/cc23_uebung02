# Lab 02: Deployment Pipelines as Code

> __Hinweis:__ Dieses Lab ist so gestaltet, dass es problemlos mit dem [Gitlab](https://git.mylab.th-luebeck.de)-Service der TH Lübeck bearbeitet werden kann.
Wer mag, kann dieses Lab aber auch mittels des Managed Service [Gitlab.com](https://gitlab.com) nachvollziehen. Hierzu müssen Sie sich allerdings [registrieren](https://gitlab.com/users/sign_up). Dies betrifft insbesondere Personen, die keine Studierenden oder Angehörigen der TH Lübeck sind.

Deployment Pipelines sind ein wesentlicher Baustein im DevOps Ansatz, um Entwicklungszyklen schnell und agil zu halten. Ziel ist es, Code der in ein Code Repository eingebracht wird, möglichst automatisiert zu integrieren, bauen, testen sowie ggf. in eine Umgebung (häufig Test, Staging, Production) auszubringen.

Mit jedem Code Push wird also automatisiert geprüft, ob der Code in die bestehende Codebasis integriert werden kann, compilierbar ist, alle Tests passiert und deploybar ist. Auf diese Weise können nur funktionierende Softwarezustände in funktionierende Softwarezustände überführt werden. Entwickler sind so nicht einmal in der Lage Code zu erzeugen, der nicht automatisiert durch die Deployment Pipeline verarbeitbar ist.

Gemäß dem Everything as Code Ansatz versucht man auch Deployment Pipelines als versionierbaren Code ausdrücken zu können. Es gibt diverse solcher Managed oder Self-hosted Services, die als kommerzielle oder auch als Open Source Software genutzt werden können. Z.B.:

- GitLab CI
- Circle CI
- Travis CI
- Jenkins
- Bitbucket Pipelines
- und viele mehr

Da Gitlab als Open Source Lösung einfach installiert werden kann, werden wir das Prinzip einer Deployment Pipeline as Code am Typvertreter Gitlab CI demonstrieren. Die Ansätze anderer CI/CD Dienste funktionieren aber nach sehr vergleichbaren Konzepten. Die Wahl auf Gitlab CI als Typvertreter erfolgt schlicht und ergreifend auf Basis der guten Verfügbarkeit von Gitlab als Open Source Software und dessen häufigen Einsatz in Cloud-native Kontexten.

## Inhalt

- [Lab 02: Deployment Pipelines as Code](#lab-02-deployment-pipelines-as-code)
  - [Inhalt](#inhalt)
  - [Vorbereitung](#vorbereitung)
  - [Übung 1: Erzeugung von Deployment Pipelines](#übung-1-erzeugung-von-deployment-pipelines)
  - [Übung 2: Weiterreichen von Job Erzeugnissen (Artifacts)](#übung-2-weiterreichen-von-job-erzeugnissen-artifacts)
  - [Übung 3: Informationen in eine Pipeline mittels Umgebungsvariablen geben](#übung-3-informationen-in-eine-pipeline-mittels-umgebungsvariablen-geben)
  - [Übung 4: Nutzung von Images](#übung-4-nutzung-von-images)
  - [Was sollten Sie mitnehmen](#was-sollten-sie-mitnehmen)
  - [Links](#links)

## Vorbereitung

[Forken](https://git.mylab.th-luebeck.de/cloud-native/lab-gitlab/-/forks/new) Sie bitte dieses Repository in Gitlab in Ihren Namensraum.

## Übung 1: Erzeugung von Deployment Pipelines

Eine Deployment Pipeline besteht aus einer Sequenz von Stages. Jede Stage kann ein oder mehrere Jobs haben. Alle Jobs innerhalb einer Stage werden parallel und isoliert voneinander ausgeführt. Eine Stage wird nur dann ausgeführt, wenn alle Jobs der vorherigen Stage erfolgreich ausgeführt werden konnten.

Eine typische Pipeline umfasst häufig die folgenden Stages (grundsätzlich können Pipelines beliebig aussehen, es bietet sich jedoch an bewährten Pipeline Blueprints zu folgen):

- build (zum Erzeugen von Executables)
- test (zum Testen von Executables)
- deploy (zum Ausbringen von Executables)

Solch eine einfache Deployment Pipeline wollen wir nun bauen. Führen Sie hierzu bitte die folgenden Schritte aus:

__Aufgaben:__

1. Sie finden in diesem geforkten Repository eine leere `.gitlab-ci.yml` Datei an. Diese Datei definiert Ihre Pipeline, die Gitlab mit jedem Push in das Repository automatisch anstößt.
2. Fügen Sie in diese Datei nun bitte folgende Inhalte ein und committen+pushen Sie `.gitlab-ci.yml` in das Repository:

    ```yaml
    stages:
        - build
        - test
        - deploy

    job1:
        stage: build
        script:
            - echo "Hello I am job 1"

    job2:
        stage: build
        script:
            - echo "Hello I am job 2"

    job3:
        stage: test
        script:
            - echo "Hello I am job 3"

    job4:
        stage: deploy
        script:
            - echo "Hello I am job 4"
    ```
3. Gitlab führt dann automatisch, die so definierte [Pipeline](../../../pipelines) aus.
   
   ![Pipeline](pipeline.png)
4. Klicken Sie auf einen dieser Jobs, dann erhalten Sie den Konsolenoutput des Jobs.
   
   ![Job console output](job-console.png)

Eine Pipeline ist also sehr einfach mit einer YAML Datei definierbar. YAML Dateien wiederum sind gut durch Code Versionssysteme versionierbar.

Das ist eigentlich auch schon das wesentliche Prinzip von einer Deployment Pipeline as Code. Sie sehen an diesem Beispiel allerdings auch bereits weitere Aspekte die typisch für Cloud-native Deployment Ansätze sind.

- Jobs sind eigentlich nichts weiter als Shellskripte, die in einem isolierten Container ausgeführt werden.
- Können alle Jobs einer Stage erfolgreich ausgeführt werden, (exit code == 0) werden die Jobs der nächsten Stage gestartet.
- Schlägt ein Job fehl (exit code != 0), wird die nächste Stage nicht gestartet. Sie können das ganz einfach ausprobieren, indem Sie bspw. den Befehl `exit 1` in *job3* ergänzen.
    ```yaml
    job3:
        stage: test
        script:
            - echo "Hello I am job 3"
            - exit 1
    ```
    Die Pipeline schlägt dann in job3 in Stage `test` fehl.

    ![Pipeline job failed](pipeline-job-failed.png)

## Übung 2: Weiterreichen von Job Erzeugnissen (Artifacts)

Jobs laufen isoliert in einem Container ab, sind also zustandslos oder anders ausgedrückt: Jobs "vergessen" erzeugte Artifakte. Dies ist sicherlich in vielen Fällen nicht sinnvoll. Z.B. sollten durch den Compiler erzeugte `.class` Dateien in einem Java Build Schritt an einen Test Job weitergereicht werden können (ansonsten müsste der Test Job erneut kompilieren). Zum Ende der Pipeline soll vielleicht auch eine `jar`-Datei als Endergebnis der Pipeline bereitgestellt werden können.

Hierfür dienen in Pipelines sogenannte Artefakte. Artefakte sind Job Erzeugnisse, die zwischen Jobs entlang einer Stage Sequenz fließen.

__Aufgabe__:

Ändern Sie bitte Ihre Pipeline wie folgt ab:

```yaml
stages:
    - generate
    - consume

job1:
    stage: generate
    script:
        - mkdir build
        - echo "Hello I am job 1" > build/job1-result.txt

job2:
    stage: generate
    script:
        - mkdir build
        - echo "Hello I am job 2" > build/job2-result.txt

job3:
    stage: consume
    script:
        - cat build/*-result.txt
```

Die Jobs job1 und job2 lenken ihre Resultate also in zwei Dateien um, die im `build` Verzeichnis gespeichert werden. Wir würden an dieser Stelle erwarten, dass der job3 daher folgende Konsolenausgabe erzeugen sollte:

```
Hello I am job1
Hello I am job2
```

Tatsächlich schlägt der job3 aber wie folgt fehl:

```
cat: can't open 'build/*-result.txt': No such file or directory
$ cat build/*-result.txt
ERROR: Job failed: exit code 1
```

Und dies obwohl die Dateien in job1 und job2 korrekt angelegt wurden. Allerdings in einem Container. Und alle Jobs laufen in isolierten Containern voneinander ab, d.h. job1 kennt job2 und job3 nicht, und job2 kenn job1 sowie job3 nicht, usw.. Wenn man Erzeugnisse eines Jobs anderen Jobs innerhalb einer Pipeline bereitstellen muss, dann kann man dies mittels Artefakten machen.

Um Artefakte zu kennzeichnen, können Sie folgenden Eintrag den Jobs job1 und job2 hinzufügen.

```yaml
artifacts:
    paths:
    - build/
```

Diese Artefakte stehen dann allen Jobs in der Pipeline zur Verfügung. Sie müssen darauf achten, dass unterschiedliche Jobs unterschiedlich benannte Artefakte erzeugen, ansonsten überschreiben sich identisch benannte Artefakte gegenseitig.

Auch dies können Sie einmal ausprobieren, indem Sie anstelle von 

```
echo "Hello I am job 2" > build/job2-result.txt
```

folgendes schreiben (also die Job-Nummern in den Artefaktbezeichnern sowohl in Job1 als auch in Job2 entfernen).

```
echo "Hello I am job 2" > build/job-result.txt
```

Dann werden Sie nur eine Ausgabe von job1 oder job2 bekommen. Welche Ausgabe ist davon abhängig welche Job Artifakte von der Pipeline aus job1 und job2 als letztes gesichert wurden. Gehen Sie davon aus, dass dies nicht deterministisch ist - insbesondere bei parallel blaufenden Jobs.

Weiteres zum Artefakt-Handling finden Sie [hier](https://docs.gitlab.com/ee/ci/pipelines/job_artifacts.html).

## Übung 3: Informationen in eine Pipeline mittels Umgebungsvariablen geben

Jobs innerhalb von Pipelines benötigen ggf. weitere Informationen. Gem. den 12 Factor Prinzipien kann man diese den Containern als Umgebungsvariablen bereitstellen. Gitlab selber hat eine ganze Reihe von [vordefinierten Umgebungsvariablen](https://docs.gitlab.com/ee/ci/variables/predefined_variables.html), die man auswerten kann, um die Pipeline zu steuern. Z.B. könnte ein Build auf dem Master-Branch grundsätzlich in die Production Umgebung deployt werden, ein Build auf dem release Branch in die Staging Umgebung und alle anderen Branches nur in die Test Umgebung.

Sie können Informationen mittels Umgebungsvariablen im Wesentlichen auf die folgenden Arten an Jobs innerhalb von Pipelines übergeben:

1. Mittels Pipeline globaler Variablen indem Sie diese Variablen Toplevel in der `.gitlab-ci.yml` deklarieren. Also bspw. so:
   ```yaml
   variables:
      FOO: "BAR"
   ```
2. Mittels der bereits erwähnten [vordefinierten Umgebungsvariablen](https://docs.gitlab.com/ee/ci/variables/predefined_variables.html), die vom Gitlab CI Build-System gesetzt werden.
3. Mittels in der Gitlab CI Oberfläche gesetzten Variablen. Diese können Sie in den [CI/CD Settings](../../../settings/ci_cd) (Variables) eines jeden Repositories setzen. Dies bietet sich insbesondere für Daten an, die niemals in eine Versionsverwaltung gehören - also insbesondere Zugangsdaten, Passwörter. Diese Variablen können in der Gitlab CI Oberfläche sogar als Masked gekennzeichnet werden, damit diese Werte nicht in Logdateien oder Konsolenausgaben im Klartext zu lesen sind.

Alle diese Variablen werden in Job Containern als Umgebungsvariablen gesetzt und können mittels der Standard Shell Variableninterpolation `$FOO` ausgewertet werden bzw. von Programmen ausgelesen werden. Häufig nutzt man dies dazu den Pipeline Prozess zu steuern. Dies soll dieses Beispiel veranschaulichen.

__Aufgabe:__

Ändern Sie die Pipeline nun bitte wie folgt ab:

```yaml
stages:
    - generate
    - consume

job1:
    stage: generate
    script:
        - mkdir build
        - echo "Hello I am job 1 executed on the $CI_COMMIT_REF_NAME branch only" > build/job1-result.txt
    artifacts:
        paths:
            - build/
    only:
        - master

job2:
    stage: generate
    script:
        - mkdir build
        - echo "Hello I am job 2 executed on the $CI_COMMIT_REF_NAME branch only" > build/job2-result.txt
    artifacts:
        paths:
            - build/
    only:
        variables:
            - $CI_COMMIT_REF_NAME == "release"

job3:
    stage: generate
    script:
        - mkdir build
        - echo "Hello I am job3 and always executed except for the master or release branch" > build/job3-result.txt
    artifacts:
        paths:
            - build/
    except:
        - master
        - release
            
job4:
    stage: consume
    script:
        - cat build/*-result.txt 
```

Dies erweitert die Pipeline um einen weiteren Job in der generate Stage. Jobs werden nun aber abhängig von Umgebungsvariablen ausgeführt.

- job1 wird nur auf dem master Branch ausgeführt.
- job2 wird nur auf dem release Branch ausgeführt.
- job3 wird auf allen anderen Branches ausgeführt.

Hierzu wurden `only` bzw. `except` den Jobs als Bedingung mitgegeben. 

- [`only`](https://docs.gitlab.com/ee/ci/yaml/#onlyexcept-basic) schließt einen Job ein, wenn alle Schlüssel (bzw. einer der Branch- oder Tag-Bezeichner) mindestens eine übereinstimmende Bedingung (bzw. eine Übereinstimmung) haben.
- [`except`](https://docs.gitlab.com/ee/ci/yaml/#onlyexcept-basic) schließt einen Job aus, wenn einer der Schlüssel (bzw. einer der Branch- oder Tag-Bezeichner) mindestens eine übereinstimmende Bedingung (bzw. eine Übereinstimmung) hat.

Veranschaulichen Sie sich die Wirkungsweise:

1. Pushen Sie dieses Pipeline einmal in den master Branch erhalten Sie die Ausgabe im job4
    ```
    Hello I am job 1, executed on the master branch only.
    ```
2. Pushen Sie diese Pipeline in den release Branch erhalten Sie die Ausgabe im job4.
    ```
    Hello I am job 2, executed on the release branch only.
    ```
3. Pushen Sie diese Pipeline in irgendeinen anderen Branch erhalten Sie die Ausgabe im job4.
    ```
    Hello I am job3, and always executed except for the master or release branch
    ```

Auf diese Weise lassen sich einzelne Jobs in der Pipeline nur unter Bedingungen ausführen, die sich über Umgebungsvariablen setzen lassen.

## Übung 4: Nutzung von Images

Bislang haben wir im Wesentlichen nur Kommandozeilen Programme des Linux/UNIX Standardumfangs genutzt. Wir wollen nun zwei kleine Hello-World Programme in Java und Python bauen, um zu demonstrieren, dass man in Jobs unterschiedliche Images für Jobs nutzen kann.

__Aufgaben:__

Sie finden hierzu im Ordner `src` die beiden folgenden Dateien (wenn nicht, legen Sie diese bitte mit den angegebenen Namen `Hello.java` und `hello.py` an):

__`Hello.java`:__
```java
public class Hello {
    public static void main(String[] args) {
        System.out.println("Hello " + args[0]);
    }
}
```

__`hello.py`:__
```python
import sys
print(f"Hello {sys.argv[1]}")
```

Passen Sie dann bitte Ihre `.gitlab-ci.yml`-Datei wie folgt an:

```yaml
variables:
    GREET: "Mundo"

stages:
    - test

java:
    stage: test
    script:
        - javac src/*.java
        - java -cp src/ Hello $GREET > result.txt
        - cat result.txt
        - cat result.txt | grep "Hello $GREET"

python:
    stage: test
    script:
        - python src/hello.py $GREET > result.txt
        - cat result.txt
        - cat result.txt | grep "Hello $GREET"
```

Wenn Sie diese Pipeline laufen lassen, werden Sie folgende Fehlermeldungen im Job *java* bzw. *python* bekommen:

```
javac: not found
python: not found
```

Dies hängt damit zusammen, dass das Standard Gitlab Job Image weder das Java JDK noch die Python Laufzeitumgebung beinhaltet. Sie können Jobs aber auf Basis anderer Images laufen lassen, z.B. mit den Standard Images, die auf DockerHub für die gebräuchlichsten Programmiersprachen angeboten werden. 

Sie können z.B.

- die Angaben `image: "openjdk:14"` im Job *java* hinzufügen, um vorzugeben, dass der *java* Job auf dem openjdk Image für Java 14
- und die Angabe `image: "python:3-slim"` im Job *python* hinzufügen, um vorzugeben, dass der *python* Job auf dem Python 3 slim (besonders kleines Python3 Image)

basieren soll.

Wenn Sie diese Angaben ergänzen, werden Sie sehen, dass die Pipeline nun durchläuft. Auf diese Weise können Sie also an unterschiedlichen Stellen in einer Pipeline unterschiedliche Container Images nutzen. Idealerweise sollten die Images natürlich kompatibel zur beabsichtigten Production Umgebung sein.

Wenn alle (oder viele) Jobs einer Pipeline auf demselben Image basieren sollen, können Sie diese `image` Angabe auch außerhalb der Jobs als Default Job Image der Pipeline angeben. Sie müssen dann nur noch bei den Jobs andere Images angeben, die explizit nicht mit dem Default Job Image der Pipeline laufen sollen.

## Was sollten Sie mitnehmen

1. Deployment Pipelines sind ein wesentlicher Baustein im DevOps Ansatz, um Entwicklungszyklen schnell und agil zu halten.
2. Gemäß dem Everything as Code Ansatz versucht man auch Deployment Pipelines als versionierbaren Code ausdrücken zu können.
3. Deployment Pipelines gliedern sich in eine Folge sequentiell ausgeführter Stages (o. ähnl. Bezeichnung).
4. Innerhalb von Stages laufen parallel ausführbare Jobs (o. ähnl. Bezeichnung).
5. Jeder Job läuft in einem Container, nicht alle Jobs müssen dasselbe Container-Image haben.
6. Ein Job ist ein Shellskript (Sequenz von Commands), dass erfolgreich ist, wenn es den Exit Code 0 liefert. Wenn ein Command einen anderen Exit Code als 0 liefert, wird der Job als nicht erfolgreich abgebrochen.
7. Das Repository wird in jedem Job als Input bereitgestellt.
8. Ein Job kann Artefakte (Output) produzieren, die an Jobs in Folge-Stages weitergegeben und auch archiviert werden können.
9. Schlägt ein Job fehl, schlägt die Pipeline fehl.
10. Gitlab bietet mit `.gitlab-ci.yml` eine einfache Möglichkeit an, solche Deployment Pipelins als YAML-Dateien zu definieren und zu versionieren.

## Links

- Gitlab: [Job Artifacts](https://docs.gitlab.com/ee/ci/pipelines/job_artifacts.html)
- Gitlab: [Predefined Variables](https://docs.gitlab.com/ee/ci/variables/predefined_variables.html)
- Youtube: [Gitlab CI pipeline tutorial for beginners](https://youtu.be/Jav4vbUrqII)
- Youtube: [Gitlab CI Pipeline, Artifacts and Environments](https://youtu.be/PCKDICEe10s)
