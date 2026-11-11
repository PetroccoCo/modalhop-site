# modalhop-site

Static placeholder for <https://modalhop.com>: a single "ModalHop is in development" page. Plain HTML and CSS, no build step, no scripts, no tracking.

The ModalHop application is developed in a separate private repository. Nothing from it lives here, and this site does not host any backend or API.

## Publishing source

GitHub Pages, branch-based: branch `main`, folder `/` (root). There is no GitHub Actions workflow. The custom domain is set in repo Settings → Pages and recorded in the `CNAME` file.

## Updating the page

1. Branch from `main`, edit `index.html` or `styles.css`, open a pull request.
2. A human reviews and merges. Pages republishes within a minute or two.
3. Never commit secrets, registrar details or private infrastructure details.

## Domain verification and DNS

DNS stays at Porkbun (registrar and nameservers do not change). Only these records are managed for this site; leave every other record (mail, SPF, others) alone.

1. **Verify the domain first.** Add the TXT record GitHub shows under your account's Settings → Pages → Add a domain, at host `_github-pages-challenge-<account>` with the value GitHub gives you. Confirm GitHub shows the domain as verified before adding any web record.
2. Apex `modalhop.com`: `A` records to `185.199.108.153`, `185.199.109.153`, `185.199.110.153`, `185.199.111.153`, and optionally `AAAA` records to `2606:50c0:8000::153`, `2606:50c0:8001::153`, `2606:50c0:8002::153`, `2606:50c0:8003::153`. Replace any registrar parking records on the apex rather than adding alongside them.
3. `www.modalhop.com`: `CNAME` to `petroccoco.github.io`. Replace any parking record on `www`.
4. Set the custom domain `modalhop.com` in Settings → Pages (this keeps `CNAME` in sync). Pages then redirects `www` to the apex.

Record TTL is the Porkbun default (600 seconds) unless noted here.

## HTTPS

After DNS settles, GitHub provisions a certificate (can take up to about an hour). Then tick "Enforce HTTPS" in Settings → Pages. Check, with no certificate warnings and from a network that is not on a private VPN:

```sh
dig +short modalhop.com A
dig +short www.modalhop.com CNAME
curl -sSIL http://modalhop.com | grep -iE '^(HTTP|location)'
curl -sSIL http://www.modalhop.com | grep -iE '^(HTTP|location)'
curl -sSI https://www.modalhop.com | head -1
curl -sSI https://modalhop.com | head -1
curl -sSI https://modalhop.com/styles.css | head -1
```

Expected: the apex serves 200 over HTTPS, `www` and plain HTTP redirect to `https://modalhop.com/`, and the stylesheet returns 200. Until the certificate is issued, treat HTTPS as pending, not done.

## Detach and roll back safely

Do these in order. Do not delete the repository or disable Pages while DNS still points at GitHub: a dangling record can let someone else claim the domain.

1. Delete or repoint the apex `A`/`AAAA` and `www` `CNAME` records at Porkbun.
2. Check with `dig` against a public resolver that they no longer point at GitHub, then wait at least the record TTL (600 seconds by default) so cached answers expire.
3. Remove the custom domain in Settings → Pages and delete the `CNAME` file.
4. Remove the GitHub verification TXT record last. Keep it until steps 1 to 3 are done, because a verified domain stops anyone else claiming it.
