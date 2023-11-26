
## Static and Dynamic Security Analysis with ScanSuite 

ScanSuite is the self contained vulnerability scanning orchestrator for the  code (SAST), Infrastructure as Code (IACS), Dependency (SCA / OSS), Dynamic Analysis (DAST) as well as Infrastructure assessment security tools.

Results are exported to [DefectDojo](https://github.com/DefectDojo/django-DefectDojo). 

Docker is also required here as many of the tools used are dockerised, others will be kindly installed as you invoke them (tested on Ubuntu).

The cli tool works well for both standalone checks and CI/CD pipeline. Here is the one of the implementation ways [Practical DevSecOps. Challenges of implementation.](https://github.com/cepxeo/presentations/blob/master/Practical_DevSecOps.pdf)

![](img/scansuite.png)

### Install DefectDojo

Execute the following command:

```
scansuite install_dojo
```
Admin password will be printed on the screen at the end of installation

### Set up connection with DefectDojo

Locate the api key by visiting https://your-defectdojo-host:8443/api/key-v2

Set environment variables:

```
export DOJO_HOST=https://your-defectdojo-host
export DOJO_APIKEY=your-api-v2-key-value
```

### Create a new Product in DefectDojo

```
scansuite init_product <App Name>
Example: ~/scansuite init_product SomeCoolApp
```

Once created, take a note of `Engagement ID`. You'll need to provide it during the scans.

### SAST scanners:

Start the scan from the source code folder.

```
cd SomeCoolApp
scansuite <scanner name> <Engagement id> 
Example: ~/scansuite semgrep 3
```
Here the `scanner name` is the keyword. Choose from the one of the following:

* semgrep     - complete ruleset for [many languages](https://semgrep.dev/docs/supported-languages/)
* semgrep_owasp - OWASP Top 10 related rules
* semgrep_gitlab - GitLab managed rules
* python      - Bandit Python code scan
* php         - PHP CS security-audit
* csharp      - Security Code Scan
* nodejs      - NodeJsScan
* ruby        - Brakeman Ruby scan
* mobsf       - Android/ Kotlin
* cscan       - Flawfinder C/C++
* gitleaks    - Detecting passwords, api keys, and tokens in git repos
* snyk - [Snyk](https://docs.snyk.io/scan-application-code/snyk-code/snyk-code-language-and-framework-support) code scan for many languages. Obtain Auth Token [here](https://app.snyk.io/account/) to use during scanner activation.
* codeql - CodeQL scanner for [many languages](https://docs.github.com/en/code-security/codeql-cli/getting-started-with-the-codeql-cli/preparing-your-code-for-codeql-analysis). Specify the language name as the last argument.

```
Example: ~/scansuite codeql 3 java
```
* findsecbugs - [FindSecBugs](https://github.com/find-sec-bugs/find-sec-bugs) Java, Kotlin and Scala code scan. Requires compiled jar as an argument.

```
Example: ~/scansuite findsecbugs 3 ~/vulnado/target/vulnado-0.0.1-SNAPSHOT.jar
```

### Dependency checks:

Start the scan from the source code folder.

```
cd SomeCoolApp
scansuite <scanner name> <Engagement id> 
```

* dep_trivy   - Trivy dependency checks
* dep_owasp   - OWASP Dependency Check
* dep_snyk    - Snyk dependency checks
* gemnasium   - Supports [many languages](https://docs.gitlab.com/ee/user/application_security/dependency_scanning/)
* retire      - Retire JS checks NodeJS/ npm dependencies.

### DAST scan:

```
scansuite <scanner name> <Engagement id> <URL>
Example: ~/scansuite zap_full 3 https://ginandjuice.shop
```

* zap_base     - ZAP quick baseline scan
* zap_full     - ZAP full scan
* arachni      - Arachni 1.5
* arachni_new  - Arachni 1.6
* nikto        - Nikto by Netsparker
* dastardly    - PortSwigger Dastardly. Results are not exported due to not supported by DefectDojo
* nuclei       - Nuclei vulnerability scanner
* wpscan       - WordPress Scanner
* sslyze       - SSL checks. Example: ~/scansuite sslyze 3 google.com:443
* scanweb   - an attempt to combine several scanners (dirsearch, zap_full, arachni_new, nuclei) against the same target

```
scanweb <Engagement id> <URL>
Example 1: ~/scanweb 3 https://ginandjuice.shop
Example 2: for site in $(cat targets.txt); do ~/scanweb 3 $site; done
```

### IACS (Infrastructure as Code) scan:

Start the scan from the source code folder.

```
cd SomeCoolApp
scansuite iacs_kics <Engagement id> 
```

* iacs_kics - Checkmarx KICS scanner for Ansible, AWS CloudFormation, Kubernetes, Terraform, Docker
* iacs_trivy - Trivy checks for config files and dependencies. Results are not exported due to not supported by DefectDojo.

### Infrastructure checks:

* nmap full scan with vulners script. Example: ~/scansuite nmap 3 192.168.3.4

* [Trivy](https://github.com/aquasecurity/trivy) Docker image scan. Requires the image name with the tag.

```
scansuite image_trivy <Engagement id> <image name>
~/scansuite image_trivy 3 vulnerables/web-dvwa:latest                  
```

* [Docker Bench for Security](https://github.com/docker/docker-bench-security) CIS Docker Benchmarks checks.

Run on the tested host:

```
~/scansuite docker_bench 3              
```

* [kube-bench](https://github.com/aquasecurity/kube-bench) CIS Kubernetes Benchmarks checks.

Run on the tested host:

```
~/scansuite kube_bench 3              
```

* [Open SCAP](https://www.open-scap.org/security-policies/scap-security-guide/) Infrastructure Benchmarks checks.

For Ubuntu and RHEL checks against CIS L1 execute on the tested host:

```
scansuite oscap_ubuntu <Engagement id> <Ubuntu (16, 18, 20 or 22) or RHEL (7,8,9) version>
~/scansuite oscap_ubuntu 3 22
~/scansuite oscap_rhel 3 9
```

* [Open SCAP](https://www.open-scap.org/tools/openscap-base/) Infrastructure Vulnerability / Oval checks.

For Ubuntu and RHEL checks execute on the tested host:

```
scansuite oscap_ubuntu_vuln
~/scansuite oscap_ubuntu_vuln

scansuite oscap_rhel_vuln <RHEL (7,8,9) version>
~/scansuite oscap_rhel_vuln 9
```

Results are not exported due to not supported by DefectDojo. Check the results in saved oscap html report.

## Manual interaction with DefectDojo

Optionally one can interact with DefectDojo directly using the commands below.

### Adding manual findings to DefectDojo

Add the new test to an engagement:

```
scansuite add_test <Engagement id> <Test name>
```

Add the new finding to the test:

```
scansuite add_finding <Test id> <Finding name> <Severity> <Description> <Mitigation>

Example:
scansuite add_finding 368 "Sensitive data exposure" Medium "/api/application.wadl is accessible" "Delete the file ot restrict an access to it"
```

### Creating additional engagement for the product:

Take a note of "id" of the product. It's in the URL when browsing the product. 
Create a new Engagement within the Product:

```
scansuite init_engage <Product id>        
Example: ~/scansuite init_engage 2
```

### Upload arbitrary report to DefectDojo

Supported report formats could be found [Here](https://defectdojo.github.io/django-DefectDojo/integrations/parsers/)

```
scansuite export_report <Engagement id> <Test type in DefectDojo> <filename>
./scansuite export_report 1 "OpenVAS CSV" openvas-test.csv
```

## DevSecOps like a pro

### Install GitLab

It is pretty resource consuming, better run it on separate host:

```
scansuite install_gitlab
```

Admin password will be printed on the screen at the end of installation
It will be running on https://YOUR_IP

### Register a GitLab worker

* Create a new runner via `https://your-gitlab-host/admin/runners/new`
* Add your-gitlab-host name/ip to /etc/hosts if not known via DNS
* Execute: `scansuite install_runner`

### Configure pipeline template

Create or import the project in GitLab. Add `.gitlab-ci.yml` in the sources root folder. Example can be taken from `example-pipelines` folder of this repository.