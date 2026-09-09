# talay-environments

Argo CD'nin uygulamalar için okuduğu desired-state reposudur. Secret değerleri burada bulunmaz; yalnızca Vault path referansları bulunur.

Yeni uygulama ekleme:

1. `examples/<environment>/<app>` örneğini `apps/<environment>/<app>` altına kopyalayın.
2. Image repository/tag, domain, namespace ve Vault `remoteKey` değerlerini gerçek değerlerle değiştirin.
3. `app.yaml` içindeki `valuesFile` yolunu doğrulayın.
4. PR review sonrası merge edin; Argo CD prune/self-heal ile uygular.

`apps/*/*/app.yaml` dosyaları aktif deploy edilir. `examples/` hiçbir zaman ApplicationSet tarafından okunmaz; bu nedenle placeholder image/domain yanlışlıkla cluster'a gitmez.

Image tag olarak `latest` kullanmayın. CI image digest veya immutable `sha-<commit>` tag'ını bu repoya PR ile yazmalıdır. Prod değişiklikleri branch protection, zorunlu review ve environment approval ile korunmalıdır.

## Finance Follower activation prerequisites

`apps/prod/finance-follower-*` manifestleri Talay chart'ları üzerinden TLS ingress, non-root/read-only
container, Vault ExternalSecret, GHCR pull secret, NetworkPolicy, HPA/PDB ve gözlemlenebilirlik
sözleşmelerini uygular. Merge etmeden önce aşağıdakiler tamamlanmalıdır:

1. `finance.cantalay.com`, `finance-admin.cantalay.com` ve `finance-api.cantalay.com` DNS A
   kayıtlarını Traefik public IP'sine yönlendirin.
2. `kv/apps/finance-follower/backend` secret'ını oluşturun.
3. `finance-follower` Keycloak realm'i ile üç OIDC client'ını Terraform üzerinden uygulayın.
4. Manifestlerdeki `sha-*` image etiketlerinin GHCR'da yayınlandığını doğrulayın.

Bu ön koşullar tamamlanmadan branch'i `main`e merge etmek Argo CD'nin eksik secret/identity
bağımlılıklarıyla workload başlatmasına neden olur.

## Todogi migration preparation

`examples/prod/auth-gateway`, `examples/prod/todogi-backend` ve `examples/prod/todogi-web`; mevcut cluster'dan çıkarılan image, domain, port, Vault key ve health endpoint sözleşmesini taşır. Backend ve gateway yeni topolojiye uyarlanmış `apps/todogi/*` Vault yollarını kullanır. `auth.cantalay.com/auth/*` gateway'e, aynı hosttaki `/realms/*` yolları Keycloak'a gider. Private GHCR erişimi `platform/registry/ghcr` Vault yolundan namespace-scope `ghcr-pull` Secret'ına çevrilir. Üç image non-root üretildikten ve immutable tag/digest değerleri yazıldıktan sonra bu örnekler `apps/` altına taşınır.

Todogi cutover öncesinde ayrıca `todogi.singlestranger.com`, `www.todogi.singlestranger.com`, `api.singlestranger.com` ve `www.api.singlestranger.com` DNS kayıtları yeni Traefik adresine yönlendirilmelidir. Hedef PostgreSQL'de secret sözleşmesindeki database/role/schema ve gerekli veri geri yüklemesi doğrulanmadan backend aktive edilmemelidir.
