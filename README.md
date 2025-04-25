# GitLab Self-Hosted Setup with Docker Compose

This README provides instructions for setting up a self-hosted GitLab instance using Docker Compose.

## Prerequisites

- Docker and Docker Compose installed on your system
- Basic understanding of Docker containers
- Sufficient system resources (recommended: 4+ CPU cores, 8+ GB RAM, 50+ GB storage)
- A server or machine with ports 8929, 443, and 2424 available

## Environment Setup

1. Create a directory for your GitLab installation:

```bash
mkdir -p ~/gitlab
cd ~/gitlab
```

2. Create a `.env` file to define the GitLab home directory:

```bash
echo "GITLAB_HOME=$(pwd)" > .env
```

3. Create a `docker-compose.yml` file with the following content:

```yaml
services:
  gitlab:
    image: gitlab/gitlab-ee:16.11.0-ee.0
    container_name: gitlab
    restart: always
    hostname: 'gitlab.example.com'
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://gitlab.example.com:8929'
        gitlab_rails['gitlab_shell_ssh_port'] = 2424
    ports:
      - '8929:8929'
      - '443:443'
      - '2424:22'
    volumes:
      - '$GITLAB_HOME/config:/etc/gitlab'
      - '$GITLAB_HOME/logs:/var/log/gitlab'
      - '$GITLAB_HOME/data:/var/opt/gitlab'
    shm_size: '256m'
```

## Configuration Details

- **Image**: Using GitLab Enterprise Edition version 16.11.0-ee.0
- **Hostname**: Set to `gitlab.example.com` (change as needed)
- **External URL**: Set to `http://gitlab.example.com:8929` (change as needed)
- **Ports**:
  - Web UI: 8929
  - HTTPS: 443
  - SSH: 2424 (mapped to internal port 22)
- **Volumes**:
  - Configuration files: `$GITLAB_HOME/config`
  - Log files: `$GITLAB_HOME/logs`
  - Data: `$GITLAB_HOME/data`
- **Shared memory**: Set to 256MB

## Starting GitLab

1. Launch GitLab with Docker Compose:

```bash
docker-compose up -d
```

2. Monitor the startup process:

```bash
docker-compose logs -f
```

GitLab may take a few minutes to fully initialize on the first run.

## Accessing GitLab

1. Once GitLab is running, access it in your browser at:
   - `http://gitlab.example.com:8929` (if hostname is configured in your DNS or hosts file)
   - `http://localhost:8929` (if running locally)

2. For the first login, use the username `root` and the automatically generated password.
   You can retrieve the initial root password with:

```bash
docker exec -it gitlab grep 'Password:' /etc/gitlab/initial_root_password
```

**Note**: This initial password is valid for 24 hours after installation. Make sure to change it immediately after your first login.

## Domain Configuration

For a production environment, you should:

1. Replace `gitlab.example.com` with your actual domain in the `docker-compose.yml` file
2. Configure DNS settings to point your domain to the server's IP address
3. Consider setting up HTTPS with Let's Encrypt or your own certificates

## SSH Access

GitLab's SSH functionality is available on port 2424. To use Git over SSH, you'll need to specify the port:

```bash
git clone ssh://git@gitlab.example.com:2424/username/project.git
```

## Maintenance

### Backup

GitLab automatically creates backups. To create a manual backup:

```bash
docker exec -it gitlab gitlab-backup create
```

Backups are stored in `/var/opt/gitlab/backups` inside the container, which is mapped to `$GITLAB_HOME/data/backups` on your host.

### Updates

To update GitLab to a newer version:

1. Update the image version in your `docker-compose.yml`
2. Pull the new image and restart:

```bash
docker-compose down
docker-compose pull
docker-compose up -d
```

## Troubleshooting

- **Container fails to start**: Check logs with `docker-compose logs gitlab`
- **Cannot access web UI**: Verify port mappings with `docker-compose ps`
- **Performance issues**: The default `shm_size` of 256MB might be insufficient for larger installations; increase as needed
- **Email not working**: Configure SMTP settings in the GitLab configuration

## Additional Resources

- [GitLab Docker Documentation](https://docs.gitlab.com/ee/install/docker.html)
- [GitLab Configuration Documentation](https://docs.gitlab.com/omnibus/settings/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)

## License

GitLab EE is licensed under the [GitLab Enterprise Edition License](https://about.gitlab.com/pricing/licensing-faq/).
