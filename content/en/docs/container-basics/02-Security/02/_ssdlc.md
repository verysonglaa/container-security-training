---
title: "SSDLC"
weight: 21
sectionnumber: 2.1
---

## Secure Software Development Lifecycle

Docker security covers the entire lifecycle of containers, including their runtime, build process, and orchestration. Key security areas include base images, Dockerfiles, container runtimes, and securing the Docker daemon. Additionally, it's important to configure container isolation, manage user privileges effectively, and follow security best practices when orchestrating containers at scale.

To ensure we fully understand and manage our workload, it's crucial to focus on our Software Development Lifecycle (SDLC). A key part of this is adopting a [Secure Software Development Lifecycle (SSDLC)](<https://www.hackerone.com/knowledge-center/what-ssdlc-secure-software-development-life-cycle>), which integrates security into every stage of development and deployment. One important aspect of a SSDLC is knowing exactly what components are in the software we're building and running.

## SBOMs and vulnerabilites

A Software Bill of Materials (SBOM) is a detailed list of all the components, libraries, and dependencies used in a software application. It acts like a product inventory for software, allowing developers, security teams, and stakeholders to know exactly what goes into an application. This transparency helps in identifying vulnerabilities, managing licensing risks, and ensuring compliance. With an SBOM, organizations can quickly assess the impact of security vulnerabilities or breaches, as they have a clear view of all the third-party and open-source components in their software stack.

Recent security issues like the Log4j vulnerability and the SolarWinds breach have underscored the need to know exactly what components are being used in software to mitigate risks quickly. By incorporating SBOMs into CI/CD pipelines, developers can automate the tracking of software dependencies, detect vulnerabilities earlier, and ensure compliance with security standards, reducing the chances of introducing insecure components into production.

We have several tools to track the dependencies which our application/images are using an open-source and easy to use one is [trivy](https://trivy.dev/). Let us try it out with our multi-stage image we built in the previous lab, in case you did not use the tag `v0.1` you can add it with `docker tag example-spring-boot-helloworld:YOURTAG example-spring-boot-helloworld:v0.1`:

```bash
trivy image --format spdx-json --output result.json example-spring-boot-helloworld:v0.1
```

Sometimes the trivy API gets overwhelmed with requests and reports an Error, just try again after a minute if that happens. Once done, the scanner will scan the files/image and determine which language is the application written. Once determined, it will download the database pertaining to that specific language and get the list of libraries that are present in that language and check against which are being used in the current context. We can now examine that file and examine all libraries and packes installed in the image

```bash
jq . result.json
```

Finally we can check if there are currently know vulnerabilites in these dependencies:

```bash
trivy sbom result.json
```

We created the SBOM explicitly to show the process, in reality the command can be abbreviated to a simple

```bash
trivy image example-spring-boot-helloworld:v0.1
```

As you can see, we obtain the library name, CVE vulnerability number, severity level (HIGH, MEDIUM, LOW), vulnerability status (fixed, not fixed, or will not fix), and if fixed, the version with the fix, along with detailed information about the vulnerability.

With this data, we can upgrade libraries with fixes, assess the risk level of unfixed vulnerabilities, and remove unnecessary vulnerable libraries. Additionally, we have the opportunity to explore alternative libraries that are more secure.

In our case we might find some vulnerable java libaries which need to be updated in the file `build.gradle`. If you have some experience with gradle you can try to fix it and build a new image.

In SSLDC scanning tools like this are mostly part of a mandatory step in a CI/CD Pipeline before uploading the image to a registry. Generally CVE's with a score up to a certain treshold are accepted and the rest is blocked.
