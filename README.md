# jenkins-docker

```
docker build -t goldginkgo/jenkins:<version> -f Dockerfile .
docker push goldginkgo/jenkins:<version>
git commit -am "Bump up version"
git tag -a <version> -m "version <version>"
git push origin master --tags

# Test
docker run -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home goldginkgo/jenkins:<version>
```
