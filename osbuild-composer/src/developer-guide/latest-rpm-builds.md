# Latest RPM builds

While developing osbuild and osbuild composer it is convenient to download the latest RPM builds directly from upstream. The repositories in the osbuild organization don't use any automation from Copr or Packit. Instead, the RPMs are built directly in the Jenkins CI and stored in AWS under the commit hash which allows anyone to download precisely the version built from a desired commit.

The URL is specified in the `mockbuild.sh` scripts in the osbuild and osbuild-composer repositories:
 * [mockbuild.sh in osbuild-composer](https://github.com/osbuild/osbuild-composer/blob/f091af55d89ac9e77aa34b94e0180aacead3f32e/schutzbot/mockbuild.sh#L27)
 * [mockbuild.sh in osbuild](https://github.com/osbuild/osbuild/blob/850ee4466f0e3335d4c21871a5f2549f2f571965/schutzbot/mockbuild.sh#L27)

And the final resulting URL is displayed in the Jenkins output (available only from Red Hat VPN).

*Common trap: If you click on a link to a repo, such as:*

[http://osbuild-composer-repos.s3-website.us-east-2.amazonaws.com/osbuild-composer/rhel-8.4/x86\_64/6b67ca34caf0ff9d31fabb398f50533c1e41c847/](http://osbuild-composer-repos.s3-website.us-east-2.amazonaws.com/osbuild-composer/rhel-8.4/x86\_64/6b67ca34caf0ff9d31fabb398f50533c1e41c847/)

*you will get HTTP 403 because that's a directory and we don't allow directory listing. If you append a known file path, such as* `repodata/repomd.xml` *you will see that the repo is there:*

[http://osbuild-composer-repos.s3-website.us-east-2.amazonaws.com/osbuild-composer/rhel-8.4/x86\_64/6b67ca34caf0ff9d31fabb398f50533c1e41c847/repodata/repomd.xml](http://osbuild-composer-repos.s3-website.us-east-2.amazonaws.com/osbuild-composer/rhel-8.4/x86\_64/6b67ca34caf0ff9d31fabb398f50533c1e41c847/repodata/repomd.xml)
