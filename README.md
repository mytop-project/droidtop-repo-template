# droidtop plugin catalog template

A template for publishing your own catalog of [droidtop](https://github.com/mytop-project/droidtop) plugins. A catalog is one
`index.json` file in a public GitHub repository. droidtop reads it, shows people your catalog's name, disclaimer and keys,
and lists your plugins once they accept. This repository builds and signs that file for you.

Press **Use this template** to make your own copy, then follow [Set it up](#set-it-up). The plugins themselves live in
their own repositories; [droidtop-plugin-template](https://github.com/mytop-project/droidtop-plugin-template) is the
template for those.

> ## Disclaimer
>
> This catalog is unofficial. It is not part of droidtop, and the droidtop project does not make, review or endorse it or the plugins it lists. The catalog and its plugins are provided as is, without warranty of any kind. Concerns and takedown requests: https://github.com/your-github-organisation/catalog/issues

Replace the text above with your own when you change `disclaimer.text` in `catalog.json`: the index build checks that
this README carries the same words. droidtop shows that text and asks people to accept it before it lists anything.

## How it works

1. **Your plugins.** Each plugin is a public repository in your GitHub organisation. Each release carries a
   `*.droidplugin.tar.xz` bundle whose `manifest.jws` is signed by a key issued from your root.
2. **The index.** `.github/workflows/index.yml` runs four times a day, when you start it by hand, and when a plugin
   repository asks for it. It reads every public repository in your organisation or account and lists each bundle whose signature
   verifies to your root, under your origin. It writes `index.json` (and `index.json.jws`, its signature) to `main`.
3. **droidtop.** A person adds the catalog by its address. droidtop fetches `index.json`, shows the catalog's name, an
   Unofficial mark, your disclaimer in full, your root key's fingerprint and the key of each origin it lists, and adds
   nothing until they accept. From then on it offers your plugins, checks every bundle again on the device, and refuses
   a later copy of the index that names a different root.
4. **The web page.** `.github/workflows/site.yml` builds a page of the catalog with GitHub Pages: each plugin, its
   versions, its permissions in plain words, and an Install in droidtop button.

Your plugins are always marked Unofficial in droidtop, and each one still asks the person before it runs.

## Set it up

You need a GitHub organisation (or account) for your plugins, `openssl`, and Python 3.

1. **Choose your origin.** A short lower-case name that no one else uses, such as `acme`. Every plugin id you publish
   starts with it (`acme.weather`). `droidtop` is droidtop's own and cannot be used.
2. **Make your root key, on your own computer:**

       bash tools/make_keys.sh root acme ~/acme-keys

   This writes four files: `root.key`, `root.crt`, `catalog-root.pub` and `droidtop-plugin-key.json`. **Keep `root.key`
   offline.** The leaves you make later add `root.srl`, openssl's serial counter: keep it beside `root.key`. Never commit it or put it in a
   secret: anyone with it can sign plugins as you. Commit `catalog-root.pub` to the top of this repository.
3. **Fill in `catalog.json`:** `organisation` (the organisation or personal account your plugin repositories are in), `catalog.id` (this repository, as
   `owner/repo`), `catalog.name`, `catalog.homepage`, `catalog.origin` (step 1) and `disclaimer.text`. Copy the same
   disclaimer text into the Disclaimer section above.
4. **Sign the index (recommended).** Make a catalog-signing key and put it in this repository's secrets
   (Settings > Secrets and variables > Actions):

       bash tools/make_keys.sh catalog your-org/catalog ~/acme-keys

   Put the contents of `catalog.key` in `CATALOG_SIGNING_KEY` and of `catalog.crt` in `CATALOG_SIGNING_CERT`. Without
   them the index is published unsigned. Once droidtop has seen a signed copy, it refuses an unsigned one.
5. **Give each plugin repository its own signing key:**

       bash tools/make_keys.sh plugin acme acme.weather ~/acme-keys weather

   Put `weather.key` in that repository's `PLUGIN_SIGNING_KEY` secret and `weather.crt` in `PLUGIN_SIGNING_CERT`. A key
   made for `acme.weather` signs only that plugin; give `acme.*` instead to let one repository sign several. Each plugin
   repository also commits the `droidtop-plugin-key.json` step 2 wrote and publishes it on its releases, so droidtop
   can anchor the origin when someone installs from the repository directly. Copy the file as it is: it holds only
   your origin and the public root certificate.
6. **Build the index.** Run the Catalog index workflow (Actions > Catalog index > Run workflow). It waits, and says
   so, until `catalog-root.pub` is committed and `catalog.json` is filled in.
7. **Turn on the web page:** Settings > Pages > Source: GitHub Actions.
8. **Share it.** People add the catalog in droidtop under Settings > Plugins > Get plugins > More catalogs, by its address
   (`https://github.com/<owner>/<repo>`), or with this link, with your repository's address URL-encoded at the end:

       https://droidtop.github.io/add-catalog?address=https%3A%2F%2Fgithub.com%2F<owner>%2F<repo>

   A QR code of that link works too: droidtop reads one from a photo or screenshot.

**Faster listing (optional).** A plugin repository can ask the catalog to rebuild right after it publishes a release,
instead of waiting for the schedule. [docs/CATALOG-FORMAT.md](docs/CATALOG-FORMAT.md) has the step, and the
`CATALOG_DISPATCH_TOKEN` secret it needs.

## Files

| File | What it is |
| --- | --- |
| `catalog.json` | Your catalog's name, organisation, origin and disclaimer. You edit this. |
| `catalog-root.pub` | Your root's public key. You add this once (step 2). |
| `index.json`, `index.json.jws` | The catalog and its signature. Written by the workflow; never edit them by hand. |
| `tools/build_index.py` | Builds and checks the index. `--build` (network), `--check` (offline), `--check-signature`. |
| `tools/make_keys.sh` | Makes your root, catalog-signing and plugin-signing keys. |
| `tools/sign_jws.py`, `tools/sign_jws.sh`, `tools/verify_jws.py` | Sign and verify the detached signatures droidtop reads. |
| `tools/site/` | Builds the web page. |
| `docs/CATALOG-FORMAT.md` | The format, the signature, revocation and the reindex dispatch, in full. |

## Changing keys

If your root key is lost or exposed, make a new one and commit its `catalog-root.pub`. List the old key's
fingerprint (`keySha256`) under `superseded` in `catalog.json`, so droidtop moves people who trusted the old key to the
new one and tells them the key changed. Every key the old root issued goes with it: make a new catalog-signing key
(step 4) and a new key for each plugin repository (step 5), replace their secrets, and commit the new
`droidtop-plugin-key.json` to each plugin repository before its next release. [docs/CATALOG-FORMAT.md](docs/CATALOG-FORMAT.md) describes it.

## Licence

GPL-3.0-only, as is droidtop. See [LICENSE](LICENSE).
