# Podman Quadlets for KankaCE

Another solution besides Docker and Docker Compose is to use Podman Quadlets.
They make it easy to run the container rootless, integrate tightly with systemd, and are easy to maintain.
Features like Podman secrets let you keep sensitive data out of the quadlet files, 
so you can safely back up your configuration to Git.

## Requirements
This guide assumes your server is already up and running, with a recent and
updated version of a server-suitable Linux distribution, e.g., [Rocky Linux](https://rockylinux.org/),
[Debian](https://www.debian.org/index.de.html), etc. You also need 
[Podman](https://podman.io/docs/installation).

**Recommendation:**
To get a graphical interface to manage your quads once they are deployed, you can use Cockpit with the component for Podman containers.

<details>
<summary>Podman – Rocky Linux (dnf)</summary>

Install Podman

```bash
sudo dnf -y install podman
```

Install Cockpit with the Cockpit component for Podman containers
```bash
sudo dnf -y install cockpit cockpit-podman
```

</details>

<details>
<summary>Podman – Debian (apt)</summary>

Install Podman

```bash
sudo apt -y install podman 
```

Install Cockpit with the Cockpit component for Podman containers
```bash
sudo apt -y install cockpit cockpit-podman
```

</details>

---

## Quick Start
1. Download the repository (or your own fork of this repository) 
    ```bash
    git clone https://github.com/Kanka-CE/kanka-ce-deploy-quadlets.git ~/.config/containers/systemd/kanka-ce
    ```

2. Create the Secrets
    ```bash
    printf $(openssl rand -hex 16) | podman secret create kanka-ce-mariadb-password -
    printf $(openssl rand -hex 16) | podman secret create kanka-ce-mariadb-root-password -
    printf $(openssl rand -hex 16) | podman secret create kanka-ce-meilisearch-password -
    printf "base64:$(openssl rand -base64 32)" | podman secret create kanka-ce-app-key -
    ```
> [!WARNING]
> Do not forget to back up your secrets somewhere safe, e.g., place them in your password manager.

3. Create a Font Awesome key and place it into a secret
    <details>
    <summary>Create Fontawsome kit-id </summary>

    1. Go to https://fontawesome.com/start to create an account.
    2. Create a (free) kit.
    3. Go to https://fontawesome.com/kits
        1. Select your kit
        2. Copy the ID either from the URL: `https://fontawesome.com/kits/<kit-id>/setup`
            or from the example: `<script src="https://kit.fontawesome.com/<kit-id>.js" crossorigin="anonymous"></script>`
            and paste only the <kit-id> here.

    </details>

    ```bash
    export FONTAWESOME_KIT=<kit-id>
    printf ${FONTAWESOME_KIT} | podman secret create fontawesome-key -
    ```

3. (Optional) Modify your configuration
   
    As there are no secrets in the environment file, you do not need to modify it at all.
    However, it is still recommended to adapt the settings to your liking, such as the time zone, the application name, and similar.
    Therefore, simply modify `~/.config/containers/systemd/kanka-ce/.env` based on your needs.


5.  Create the storage location
    ```bash
    export KANKACE_STORAGE_DIR=</path/to/persistent/storage>
    mkdir -p ${KANKACE_STORAGE_DIR}/{redis,meilisearch,mariadb,KankaCE}
    sed -i "s|podman-storage-kankace|${KANKACE_STORAGE_DIR}|g" ~/.config/containers/systemd/kanka-ce/*.volume
    ```

6. Apply the changes
    ```bash
    systemctl --user daemon-reload
    ```
    And allow the service to keep running after the user logs out
    ```
    loginctl enable-linger $USER
    ```

7. Run KankaCE
    ```bash
    systemctl --user start kank-ce
    ```
    
---

## More Details on Podman Quadlets

### Download the quadlet files
First, create a (private) fork of this repository, as you probably want to adapt the configuration to your needs.
Then clone your fork (in this example, we use this repository, so do not forget to adapt the repository URL):

For **system-wide** quadlets, place the folder at `/etc/containers/systemd/`
```bash
git clone https://github.com/Kanka-CE/kanka-ce-deploy-quadlets.git /etc/containers/systemd/kanka-ce
```

(Recommended) For **rootless** quadlets, place the folder at  `~/.config/containers/systemd/`
```bash
git clone https://github.com/Kanka-CE/kanka-ce-deploy-quadlets.git ~/.config/containers/systemd/kanka-ce
```

### Manage secrets
In general, Podman secrets can be created via
```bash
printf <password> | podman secret create <secret-label> -
```

or to create a secret with a randomly generated password
```bash
printf $(openssl rand -hex 16) | podman secret create <secret-label> -
```

To view a list of all secrets, run
```bash
podman secret ls
```

and the password stored in a given secret can be accessed via
```bash
podman secret inspect --showsecret <secret-label>
```

To delete a secret, run
```bash
podman secret rm <secret-label>
```

### Manage Volumes
In the setup presented in this repository, the storage will be managed in volumes.
In this configuration, the greatest benefit of volumes is that we can run the container
with a different user ID and group ID than the user that is running podman.

The volumes in this repository are configured such that the storage content is placed into the 
folder: `</path/to/persistent/storage>/<Application-Name>`. 
> [!CAUTION]
> Remember to back up the data of the volumes, especially of KankaCE and MaraDB!

Recommendation: You can back up the data of volumes via [docker-borgmatic](https://github.com/borgmatic-collective/docker-borgmatic).

To view a list of all volumes, run
```bash
podman volumes ls
```

---

## Related repositories

| Repo | What it's for |
|---|---|
| [kanka-community-edition](https://github.com/Kanka-CE/kanka-community-edition) | The patched source code of KankaCE |
| [docker-kanka-ce](https://github.com/Kanka-CE/docker-kanka-ce) | The Dockerfile used to build the Kanka CE container |
| [kanka-ce-deploy](https://github.com/Kanka-CE/kanka-ce-deploy) | Docker-compose template, and the patches to turn Kanka into KankaCE |


## License and Acknowledgements
The files in this repository are not affiliated with the official Kanka project.  

Note that Kanka itself is licensed under the [“Commons Clause” License Condition v1.0](https://github.com/owlchester/kanka/blob/develop/LICENSE).

## ❤️ Support the Official Kanka Project
If you enjoy using the Community Edition, please consider supporting the official Kanka project:
Kanka CE exists because the upstream project is amazing.
If you enjoy using Kanka or Kanka CE, please consider supporting the original creators:

💙 **Kanka Website:** https://kanka.io  
