---
title: Setting Up an Apt Package Repository
date: 2025-09-19T14:58:15-06:00
authors:
  - name: Michael Schaecher
    image: /authors/Michael.jpg
tags:
  - O.S.
  - Howto
  - Debian/Ubuntu
  - Linux
excludeSearch: false
---

In this guide I'm going to assume you have a basic understanding of the command line and how to create a simple *DEB* package. If you need help with that, check out this [Packaging software for Debian systems](https://medium.com/@flavienb/packaging-software-for-debian-systems-19562b077050) guide. To build a more complex package, requires a deeper understanding of the packaging system, which is beyond the scope of this guide.

Debian and Ubuntu use the Advanced Package Tool (APT) to manage software installation and updates. APT retrieves packages from repositories, which are servers that host collections of software packages. Setting up your own APT package repository allows you to distribute custom software or internal applications within your organization or to the public.

## Prerequisites

Before we get started, you will need the following:

- A Linux based OS capable of running apt databases repository tools. This guide assumes you are using a Debian-based system like Ubuntu or Debian itself.
- Basic knowledge of command line operations.
- A GitHub/GitLab account (if you plan to host your repository there).
- A domain name (optional, but recommended for public repositories).
- GPG knowledge (for signing packages) Release signing.

## Step 1: Install Required Tools

Some tools are required to create and manage your APT repository. Install them using the following command:

```bash
sudo apt update
sudo apt install -y dpkg-dev gnupg apt-utils
```

Don't worry if some of these packages are already installed; the command will simply ensure you have the latest versions and install any missing dependencies as needed.

## Step 2: Create the Repository Structure

Create a directory structure for your repository. This structure will include directories for different distributions and architectures.

```tree
my-apt-repo/
├── dists/
│   └── stable/
│       └── main/
│           └── binary-amd64/
├── pool/
│   └── main/
│       ├── lib{a..z}/
│       │   └── libfoo/
│       │       └── libfoo_1.0-1_amd64.deb
│       └── {a..z}/
│           └── foo/
│               └── foo_1.0-1_amd64.deb
└── keyrings/
    └── myrepo.gpg
```

In the above structure - `dists/` contains the distribution information. The `pool/` directory holds the actual `.deb` packages, organized by package name. The `keyrings/` directory will store the public key used for signing the repository. As you can see there is generic naming conventions for release versions.

For now we'll stick with keeping it simple.

```bash
mkdir -p my-apt-repo/{dists/stable/main/binary-amd64,pool/main,keyrings}

for dir in {a..z}; do
    mkdir -p my-apt-repo/pool/main/$dir
done

cd my-apt-repo
```

This command creates the necessary directory structure for your APT repository. You can adjust the paths and names according to your needs, especially if you plan to support multiple distributions or architectures and libraries.

## Step 3: Add Your Packages

Now is time to add your `.deb` packages to the `pool/` directory. You can organize them by package name or version as shown in the structure above to keep things tidy. Or if it is just a few packages, you can place them directly in the `pool/` directory. However you choose to organize them, make sure they are easily accessible.

## Step 4: Create the Package Index

Next, you need to create the package index files that APT uses to find and install packages.

```bash
apt-ftparchive packages pool/ > dists/stable/main/binary-amd64/Packages

for c in gzip xz; do
    $c -k dists/stable/main/binary-amd64/Packages
done

rm dists/stable/main/binary-amd64/Packages
```

This command generates the `Packages` file, which contains metadata about the packages in your repository. It then compresses the file using both `gzip` and `xz` for better compatibility with different APT configurations. Finally, it removes the uncompressed `Packages` file to save space.

## Step 5: Create the Release File

The `Release` file contains information about the repository, including the available distributions and architectures. Create it using the following command:

```bash
cat <<EOF > dists/stable/Release
Origin: MyRepo
Label: MyRepo
Suite: stable
Version: 1.0
Architectures: amd64
Components: main
Description: My custom APT repository
EOF
```

This command creates a `Release` file with basic information about your repository. You can customize the fields to better describe your repository. Next is add checksums to the `Release` file.

```bash
apt-ftparchive release dists/stable >> dists/stable/Release
```

At this point, you should have a `Release` file with checksums for the `Packages` files. This is the basic structure of a working APT repository. To enhance security, you should sign the `Release` file using GPG.

## Step 6: Sign the Repository (Optional but Recommended)

To sign your repository, you need to create a GPG key if you don't already have one. You can do this using the following command  and follow the prompts:

```bash
gpg --full-generate-key
```

Export your public key to a file:

```bash
gpg --armor --export "Name of Your Key" > keyrings/myrepo.gpg
```

Now sign the `Release` file:

```bash
gpg --default-key "Your Name" \
  --detach-sign -o dists/stable/Release.gpg dists/stable/Release 2> /dev/null
gpg --default-key "Your Name" \
  --clearsign -o dists/stable/InRelease dists/stable/Release 2> /dev/null
```

Replace `"Your Name"` with the name associated with your GPG key. This will create a detached signature file `Release.gpg` and a clearsigned `InRelease` file.

## Step 7: Test Your Repository Locally

To test your repository locally, you can use a simple HTTP server to serve the files. Navigate to the `my-apt-repo` directory and run:

```bash
python3 -m http.server 8000
```

Install the repository on your local machine by adding it to your APT sources list:

```bash
cat <<EOF | sudo tee /etc/apt/sources.list.d/myrepo.sources
Types: deb
URIs: http://localhost:8000
Suites: stable
Components: main
Architectures: amd64
Signed-By: /path/to/my-apt-repo/keyrings/myrepo.gpg
EOF
```

Replace `/path/to/my-apt-repo/keyrings/myrepo.gpg` with the actual path to your GPG key file.

Update your package list and install a package from your repository to verify everything is working:

```bash
sudo apt update
sudo apt install foo
```

If everything is set up correctly, APT should be able to find and install the package from your local repository. If you encounter issues with GPG signatures. The most common issue is that the public key is not trusted. Check that the `InRelease` is does contain the GPG signature.

## Step 8: Host Your Repository

One of the easiest ways to host your APT repository is by using GitHub or GitLab Pages. Both services allow you to serve static files over HTTPS, which is ideal for hosting an APT repository.

Here I will cover GitHub Pages.

1. **Create a GitHub Repository**: Create a new repository on GitHub to host your APT repository files.
2. **Push Your Repository Files**: Push the contents of your `my-apt-repo` directory to the `main` branch of your GitHub repository.

   ```bash
    git init
    git remote add origin <your-github-repo-url>
    git add .
    git commit -m "Initial commit"
    git push -u origin main
   ```

3. **Enable GitHub Pages**: Go to the settings of your GitHub repository, navigate to the "Pages" section. Under 'Build and Deployment' you well see Source. Pull down the menu and select **GitHub Actions**. The build will take a couple minutes to complete. Once done, your repository will be available at `https://<your-github-username>.github.io/<your-repo-name>`.

A basic GitHub Actions workflow using [Jykll](https://jekyllrb.com/) was created and the README.md is used as the index page. You can customize this as needed.

On the client machines that will use your repository, update the APT sources list to point to your GitHub Pages URL.

```bash
cat <<EOF | sudo tee /etc/apt/sources.list.d/myrepo.sources
Types: deb
URIs: https://<your-github-username>.github.io/<your-repo-name>
Suites: stable
Components: main
Architectures: amd64
Signed-By: /path/to/my-apt-repo/keyrings/myrepo.gpg
EOF
```

Replace `<your-github-username>` and `<your-repo-name>` with your actual GitHub username and repository name. Also, ensure the path to your GPG key file is correct.

## Conclusion

Setting up your own APT package repository allows you to distribute custom software easily. By following the steps outlined in this guide, you can create, sign, and host your repository, making it accessible to users via APT. Whether for internal use or public distribution, having your own repository can streamline software management and deployment.
