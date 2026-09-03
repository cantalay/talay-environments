# talay-environments

Argo CD'nin uygulamalar için okuduğu desired-state reposudur. Secret değerleri burada bulunmaz; yalnızca Vault path referansları bulunur.

Yeni uygulama ekleme:

1. `examples/<environment>/<app>` örneğini `apps/<environment>/<app>` altına kopyalayın.
2. Image repository/tag, domain, namespace ve Vault `remoteKey` değerlerini gerçek değerlerle değiştirin.
3. `app.yaml` içindeki `valuesFile` yolunu doğrulayın.
4. PR review sonrası merge edin; Argo CD prune/self-heal ile uygular.

`apps/*/*/app.yaml` dosyaları aktif deploy edilir. `examples/` hiçbir zaman ApplicationSet tarafından okunmaz; bu nedenle placeholder image/domain yanlışlıkla cluster'a gitmez.

Image tag olarak `latest` kullanmayın. CI image digest veya immutable `sha-<commit>` tag'ını bu repoya PR ile yazmalıdır. Prod değişiklikleri branch protection, zorunlu review ve environment approval ile korunmalıdır.
