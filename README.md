# quick-gitlab-restore
A simple script to automate restoring a GitLab Backup tar file into a new instance.


* open port 80 for LetsEncrypt. if you're using it
* script assumes you want to rebuild tables from scratch, if not, change `force=yes` to `force=no`


```shell
#set these environment variables first
SECRETS_PATH="<path to gitlab-secrets.json>" && \
INSTANCE_NAME="<gitlab container name>" && \
TAR_PATH="<path to backup tar file>"

#wait for container status to be healthy before running...


docker exec -it $INSTANCE_NAME gitlab-ctl reconfigure && \
docker exec -it $INSTANCE_NAME gitlab-rake gitlab:check SANITIZE=true && \
docker cp $TAR_PATH $INSTANCE_NAME:/var/opt/gitlab/backups && \
docker exec -it $INSTANCE_NAME gitlab-ctl stop puma && \
docker exec -it $INSTANCE_NAME gitlab-ctl stop sidekiq && \
TEMP="${GITLAB_BACKUP//"_gitlab_backup.tar"/}" && BACKUP_NAME=${TEMP##*/} && unset TEMP && \
docker exec -it $INSTANCE_NAME gitlab-backup restore BACKUP=$BACKUP_NAME force=yes && \
docker exec -it $INSTANCE_NAME rm /etc/gitlab/gitlab-secrets.json && \
docker cp $SECRETS_PATH $INSTANCE_NAME:/etc/gitlab && \
docker exec -it $INSTANCE_NAME gitlab-ctl reconfigure && \
docker restart $INSTANCE_NAME && \
unset SECRETS_PATH && \
unset INSTANCE_NAME && \
unset TAR_PATH && \
unset BACKUP_NAME

```
