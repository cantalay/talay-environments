# talay-environments

Argo CD'nin uygulamalar için okuduğu desired-state reposudur. Secret değerleri burada bulunmaz; yalnızca Vault path referansları bulunur.

Yeni uygulama ekleme:

1. `examples/<environment>/<app>` örneğini `apps/<environment>/<app>` altına kopyalayın.
2. Image repository/tag, domain, namespace ve Vault `remoteKey` değerlerini gerçek değerlerle değiştirin.
3. `app.yaml` içindeki `valuesFile` yolunu doğrulayın.
4. PR review sonrası merge edin; Argo CD prune/self-heal ile uygular.

`apps/*/*/app.yaml` dosyaları aktif deploy edilir. `examples/` hiçbir zaman ApplicationSet tarafından okunmaz; bu nedenle placeholder image/domain yanlışlıkla cluster'a gitmez.

Image tag olarak `latest` kullanmayın. CI image digest veya immutable `sha-<commit>` tag'ını bu repoya PR ile yazmalıdır. Prod değişiklikleri branch protection, zorunlu review ve environment approval ile korunmalıdır.

## Todogi migration preparation

`examples/prod/todogi-backend` ve `examples/prod/todogi-web`, mevcut cluster'dan çıkarılan image, domain, port, Vault key ve health endpoint sözleşmesini taşır. Backend yeni topolojiye uyarlanmış `apps/todogi/backend` Vault yolunu kullanır. `examples/` aktif deploy edilmez. Mevcut GHCR image'ları anonim pull'a kapalıdır ve legacy yedekte registry Secret yoktur; bu yüzden namespace-scope `imagePullSecret` sağlanmadan örnekler `apps/` altına taşınmamalıdır. Mevcut Todogi container'ları root çalıştığı için özellikle web image ayrıca non-root kullanıcı, salt-okunur filesystem ve `8080` portuyla yeniden build edilmelidir.

Todogi cutover öncesinde ayrıca `todogi.singlestranger.com`, `www.todogi.singlestranger.com`, `api.singlestranger.com` ve `www.api.singlestranger.com` DNS kayıtları yeni Traefik adresine yönlendirilmeli; PostgreSQL `todogi` schema/verisi ayrı bir backup/restore akışıyla taşınmalıdır.
