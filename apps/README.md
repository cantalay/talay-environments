# Active applications

Bu dizinde yalnızca gerçekten deploy edilecek `apps/<environment>/<project>/<component>` dizinleri bulunur.

Her component:

- `application.yaml`: Argo CD kaynak sözleşmesi
- `values.yaml`: ortak Helm chart ayarları
- `certificate/`: TLS sertifikaları
- `ingress/`: dış trafik kuralları
- `nginx/`: yalnızca web componentlerinde NGINX ayarı
- `kustomization.yaml`: component'e özel manifestlerin giriş noktası
