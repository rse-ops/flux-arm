# Spack

We will be doing this from the command line. The AMI we want is `ami-012fb2a3ce1880d5d`. Note that the prices range from $1.60 - $1.80 an hour, so about $38-$43 dollars a day.

```bash
aws ec2 describe-instance-type-offerings --location-type availability-zone --filters Name=instance-type,Values=hpc7g.* --region us-east-1 --query InstanceTypeOfferings[*].[InstanceType,Location]
```
```console
[
    [
        "hpc7g.16xlarge",
        "us-east-1a"
    ],
    [
        "hpc7g.8xlarge",
        "us-east-1a"
    ],
    [
        "hpc7g.4xlarge",
        "us-east-1a"
    ]
]
```

```bash
aws ec2 run-instances --image-id ami-0f85557b566aa0d20 --region us-east-1 --count 1 --instance-type hpc7g.16xlarge --key-name dinosaur --security-group-ids sg-0c0a804da857da410  --subnet-id subnet-0b0e2408f9960d3f6
```

Note that the subnet ID will associate you with an availability zone. In this case we needed us-east-1a (where the instance is shown to be above).

## Building

Shell into the instance with your PEM:

```bash
ssh -o 'IdentitiesOnly yes' -i path-to-key.pem ec2-user@ec2-52-55-54-40.compute-1.amazonaws.com
```

And then install docker and update:

```bash
sudo yum update -y
sudo yum install -y docker
sudo systemctl start docker
sudo usermod -aG docker $USER
sudo setfacl --modify user:ec2-user:rw /var/run/docker.sock
```

At this point give it a test!

```bash
docker run hello-world
```

If that works, we are good to build here. Let's run this in a screen because we can expect our credential to expire or otherwise get kicked off.

```bash
sudo yum install -y screen
screen
```

Clone the repository:

```bash
sudo yum install -y git
git clone -b add/spack https://github.com/rse-ops/flux-arm
cd flux-arm/spack
```

The default cpu arch is already set to arm, so we don't need to set the build arg.

```bash
$ docker buildx build --platform linux/arm64 --tag ghcr.io/rse-ops/flux-arm-spack:arm64 .
```

At this point you can tag for the date too, and push both (you'll need to add credentials to the instance)

```bash
docker tag ghcr.io/rse-ops/flux-arm-spack:arm64 ghcr.io/rse-ops/flux-arm-spack:spack-0.20.0
docker push ghcr.io/rse-ops/flux-arm-spack --all-tags
```

Finally, let's create a quick Dockerfile that adds lammps on top (this uses the image we just built above
for a base):

```bash
$ docker buildx build -f Dockerfile.lammps --platform linux/arm64 --tag ghcr.io/rse-ops/flux-arm-lammps:arm64 .
```

Also tag and push!

```bash
docker tag ghcr.io/rse-ops/flux-arm-lammps:arm64 ghcr.io/rse-ops/flux-arm-lammps:spack-0.20.0
docker push ghcr.io/rse-ops/flux-arm-lammps --all-tags
```

And that's it! You can exit and delete the instance. That was SO FAST.
