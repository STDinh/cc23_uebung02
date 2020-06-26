# Lab: Deployment Pipelines as Code

Deployment Pipelines sind ein wesentlicher Baustein im DevOps Ansatz um Entwicklungszyklen kurz und agil zu halten.
Ziel ist es, Code der in ein Code Repository eingebracht wird, möglichst automatisiert zu integrieren, bauen, testen
sowie ggf. in eine Umgebung (häufig Test, Staging, Production) auszubringen.

Gemäß dem Everything as Code Ansatz versucht man auch Deployment Pipelines als versionierbaren Code ausdrücken zu können.
Es gibt diverse solcher Managed oder Self-hosted Services die als kommerzielle oder auch als Open Source bereitstehen. Z.B.:

- GitLab CI
- Circle CI
- Travis CI
- Jenkins
- Bitbucket Pipelines
- und mehr

Da Gitlab als Open Source Lösung einfach installiert werden kann, werden wir das Prinzip einer Deployment Pipeline
as Code am Typvertreter Gitlab CI demonstrieren. Die Ansätze anderer CI/(CD Dienste funktionieren aber nach sehr
vergleichbaren Konzepten. Die Wahl auf Gitlab CI als Typvertreter erfolgt schlicht und ergreifend auf Basis der
guten Verfügbarkeit von Gitlab als Open Source Software und dessen häufigen Einsatz in Cloud-native Kontexten.

Wer mag, kann dieses Lab auch mittels des Managed Service Gitlab.com nachvollziehen. Hierzu müssen Sie sich allerdings
registrieren.

## Inhalt

- Erzeugung einer Build-Pipeline mittels einer Sequenz von Stages
- Parallele Ausführung von Jobs innerhalb einer Stage, sequentielle Ausführung der einzelnen Stages
- Weiterreichen von Artifacts zwischen Stages
- Informationen in die Build-Pipeline mittels [Environment Variables](https://docs.gitlab.com/ee/ci/variables/predefined_variables.html) geben.
- Nutzung von Images
- Bereitstellung von Images
- Deployments to Kubernetes
- Deployments to Serverless Environments

### Übung 1: Erzeugung von Buildpipelines

to be done

## Quellen für weitergehende Informationen:

- Youtube: [Gitlab CI pipeline tutorial for beginners](https://youtu.be/Jav4vbUrqII)
- Youtube: [Automating Kubernetes Deployments](https://youtu.be/wEDRfAz6_Uw)
- Youtube: [Continuous Integration with GitLab CI](https://youtu.be/EuwLdbCu3DE)


