If you are new to GitHub Actions, we’d recommend reading through [some examples](https://docs.github.com/en/actions/writing-workflows/quickstart). [](https://read.readwise.io/read/01jsef36a3c2c3sv5e69vhfq00)

## GitHub Actions
[GitHub Actions](../../Dev,%20ICT%20&%20Cybersec/Dev,%20scripting%20&%20OS/GitHub%20Actions.md) is a continuous integration and continuous delivery (CI/CD) platform that allows the execution of code specified within workflows as part of the CI/CD process [^1]. When you push code to a GitHub repository or create a pull request, GitHub Actions can automatically build, test, and deploy your code using workflows defined in YAML files. [](https://read.readwise.io/read/01jsef2pt7qmrk13eby97jf5z1)

***Every workflow run generates a GITHUB_TOKEN*** — a special, automatically generated GitHub App installation token that allows the workflow to interact with the repository. This token’s permissions can be configured in the workflow file, at the repository level, or at the org level, determining what actions it can perform within the repository. [](https://read.readwise.io/read/01jsef3v24fhsce6y5wb8g22n6)

Put simply:
- GitHub workflows execute on GitHub runners (typically a VM or Docker containers).
- GitHub runners need a way to authenticate to GitHub to do stuff the workflows tell them to do.
- For that purpose, they use the GITHUB_TOKEN [^GITHUB_TOKEN]. [](https://read.readwise.io/read/01jsef455a7b8b72y01k64cpyf)

### Mitigation against Supply Chain attacks

[Supply Chain](Supply%20Chain.md) attacks are a new attack vector. You can mitigate this vulnerabilities in this way:

- GitHub [Action Pinning](../../Readwise/Articles/Yaron%20Avital%20-%20Unpinnable%20Actions%20How%20Malicious%20Code%20Can%20Sneak%20Into%20Your%20GitHub%20Actions%20Workflows.md#Action%20Pinning) - [Want to Pin Your Actions?](Adan%20-%20GitHub%20Actions%20and%20the%20Pinning%20Problem%20What%20100%20Security%20Projects%20Reveal.md#Want%20to%20Pin%20Your%20Actions?)

[^1]: [CICD security](../../Dev,%20ICT%20&%20Cybersec/Dev,%20scripting%20&%20OS/CICD%20security.md)
[^GITHUB_TOKEN]: [Investigating Impact of exposed GITHUB_TOKEN](../../Readwise/Articles/Harry%20Hayward%20-%20CodeQLEAKED%20–%20Public%20Secrets%20Exposure%20Leads%20to%20Supply%20Chain%20Attack%20on%20GitHub%20CodeQL.md#Investigating%20Impact%20of%20exposed%20GITHUB_TOKEN)