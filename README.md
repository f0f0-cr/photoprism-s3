# Minio S3 and photoprism, both running in the same VM with containers:

## You need S3FS on the VM where you will run the containers
[https://github.com/s3fs-fuse/s3fs-fuse](https://github.com/s3fs-fuse/s3fs-fuse)

## Create Minio
```
docker run -p 9001:9001 --hostname minio --name minio -v minio:/data quay.io/minio/minio server /data --console-address ":9001"
```

## Create password file for s3fs
The default username and password for minio is minioadmin, these are also the secret and access key.
```
echo "ACCESS_KEY:SECRET_KEY" | sudo tee /home/${USER}/.passwd-s3fs
```

## Add some security
```
chmod 600 /home/${USER}/.passwd-s3fs
```

## Mount the bucket
#### Create mount point
```
mkdir /photoprism/originals
```
#### Mount the bucket
```
s3fs photoprism /photoprism/originals -o passwd_file=/home/${USER}/.passwd-s3fs,use_path_request_style,url=http://localhost:9000
```

## Create photoprism  container
```
docker rm -f photoprism;docker run -d \
  --name photoprism --hostname photoprism \
  -p 2342:2342 \
  -v /photoprism/originals:/photoprism/originals \
  -v /photoprism/storage:/photoprism/storage \
  -e PHOTOPRISM_ADMIN_PASSWORD="admin" \
  photoprism/photoprism:latest
```
