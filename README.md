# talay-environments

Argo CD'nin uygulamalar için okuduğu desired-state reposudur. Secret değerleri burada bulunmaz; yalnızca Vault path referansları bulunur.

Yeni uygulama ekleme:

1. `apps/<environment>/<project>/<component>` dizinini oluşturun.
2. `application.yaml` içinde chart, namespace, values ve manifest yollarını tanımlayın.
3. Workload ayarlarını `values.yaml`; TLS'i `certificate/`; trafiği `ingress/`; web sunucusunu `nginx/` altında yönetin.
4. PR review sonrası merge edin; Argo CD prune/self-heal ile uygular.

`apps/*/*/*/application.yaml` dosyaları aktif deploy edilir. Aynı component dizinindeki `kustomization.yaml`, uygulamaya özel Kubernetes kaynaklarını toplar.

Image tag olarak `latest` kullanmayın. CI image digest veya immutable `sha-<commit>` tag'ını bu repoya PR ile yazmalıdır. Prod değişiklikleri branch protection, zorunlu review ve environment approval ile korunmalıdır.

## Todogi

Aktif bileşenler `apps/prod/todogi/{auth-gateway,backend,web}` altındadır. Backend ve gateway `apps/todogi/*` Vault yollarını kullanır. `auth.cantalay.com/auth/*` gateway'e gider. Private GHCR erişimi `platform/registry/ghcr` Vault yolundan namespace-scope `ghcr-pull` Secret'ına çevrilir.

`todogi.singlestranger.com`, `www.todogi.singlestranger.com`, `api.singlestranger.com`, `www.api.singlestranger.com` ve `auth.cantalay.com` Traefik adresine yönlenir. Sertifikalar component dizinlerinde açık `Certificate` kaynakları olarak yönetilir.

## VitaFinder

VitaFinder `apps/prod/vitafinder/{storefront,admin,api,worker}` bileşenlerinden oluşur. Storefront ve admin aynı immutable web image'inin ayrı dizinlerini sunar; API ve worker aynı core image'ini farklı process tipiyle çalıştırır. Public adresler `vitafinder.cantalay.com`, `admin.vitafinder.cantalay.com` ve `api.vitafinder.cantalay.com` olarak Traefik adresine yönlenir. Kimlik doğrulama `auth.cantalay.com/realms/vitafinder` üzerinden yapılır.

## Finance Follower

Finance Follower `apps/prod/financefollower/{api,analytics,web}` bileşenlerinden oluşur. Web paneli `finance.cantalay.com`, API `api.finance.cantalay.com/api` adresinden sunulur; analytics yalnız cluster içidir (ingress yok). API ve analytics aynı `financefollower` PostgreSQL veritabanını ve Redis DB 2'yi (`financefollower:` öneki) kullanır; secret'lar `apps/financefollower/{api,analytics}` Vault yollarından gelir. Giriş auth-gateway üzerinden `auth.cantalay.com/auth/financefollower/*` ile yapılır; realm `auth.cantalay.com/realms/financefollower`.
