# Moved To Gitlab

Just moved my blog to GitLab. It allows me the same git-based workflow as I
enjoyed with GitHub Pages, but it also lets me use SSL with my custom domain.

I found these pages useful:

- [Hosting on GitLab.com with GitLab
  Pages](https://about.gitlab.com/2016/04/07/gitlab-pages-setup/)
- [Securing your GitLab Pages with TLS and Let's
  Encrypt](https://about.gitlab.com/2016/04/11/tutorial-securing-your-gitlab-pages-with-tls-and-letsencrypt/)

Since I'm using NixOS, the TLS instructions on GitLab didn't work. I was able to
install the [Certbot](https://certbot.eff.org/) package in the main Nix channels
and substitute `letsencrypt-auto` with `certbot` in the GitLab documentation.

I also had to make a minor change to the `.gitlab-ci.yml` example in the GitLab
documentation in order to make it copy the LetsEncrypt challenge directory to
the public GitLab Pages directory. The entirety of my file as of this writing is
as follows:

```
pages:
  stage: deploy
  script:
  - mkdir .public
  - cp -r * .public
  - cp -r .well-known .public
  - mv .public public
  artifacts:
    paths:
    - public
  only:
  - master
```

Since GitLab doesn't automatically add the `.html` extension to missing pages,
as GitHub does, I also wrote a small bash script to softlink all my HTML files
to their original names minus the extension. As of this writing:

```
dir=`pwd`
for d in $(find $dir -type d)
do
  cd "$d"
  for f in $(ls *.html 2> /dev/null)
  do
    ln_f="${f%.*}"
    if [ ! -f "$ln_f" ]; then
      ln -s "$f" "$ln_f"
    fi
  done
done
```

The entirety of my blog's source code can be found at
[https://gitlab.com/ryepdx/ryepdx.gitlab.io](https://gitlab.com/ryepdx/ryepdx.gitlab.io).
