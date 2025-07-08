A continuaci\u00f3n se presenta un manual de instalaci\u00f3n que describe los pasos b\u00e1sicos para levantar RabbitMQ, Kyverno e Istio como proxy inverso del frontend, adem\u00e1s de instalar los charts de cada servicio.

## 1. Pre-requisitos
1. Contar con acceso a un cl\u00faster de Kubernetes y tener `kubectl` configurado.
2. Disponer de [Helm](https://helm.sh/) instalado.
3. Istio debe estar desplegado en el cl\u00faster. El gateway utiliza el selector `istio: ingress`.

## 2. Creaci\u00f3n de namespaces y secretos
1. Crear los namespaces necesarios:
   ```bash
   kubectl create namespace nova
   kubectl create namespace rabbitmq
   kubectl create namespace cloudflared
   kubectl create namespace kyverno
   ```
2. Registrar el `ImagePullSecret` (por ejemplo `dmirandam-regcred`) en los namespaces donde se desplieguen los charts.

## 3. Desplegar RabbitMQ
1. Aplicar `rabbitmq/secret.yaml` para las credenciales.
2. Aplicar `rabbitmq/cluster.yaml` para crear el `RabbitmqCluster` con r\u00e9plicas y almacenamiento persistente.
3. Aplicar `rabbitmq/topology.yaml` para configurar vhosts, usuarios, permisos y colas.
   ```bash
   kubectl apply -f rabbitmq/secret.yaml
   kubectl apply -f rabbitmq/cluster.yaml
   kubectl apply -f rabbitmq/topology.yaml
   ```

## 4. Instalar Kyverno y aplicar la pol\u00edtica
1. Instalar Kyverno con Helm e indicar `kyverno/kyverno-values.yaml`:
   ```bash
   helm install kyverno kyverno/ --namespace kyverno -f kyverno/kyverno-values.yaml
   ```
2. Aplicar la pol\u00edtica de verificaci\u00f3n de firmas:
   ```bash
   kubectl apply -f kyverno/cluster-policy.yaml
   ```

## 5. Despliegue de microservicios
Para instalar cada microservicio ejecute:
   ```bash
   helm install <nombre-release> ./<carpeta-del-chart> -n nova
   ```
Personalice los valores mediante ficheros `values.yaml` o la opci\u00f3n `--set`.

Los charts disponibles son:
- `frontend`
- `api-gateway`
- `auth-service`
- `transaction-service`
- `user-product-service`
- `notification-service`

## 6. Configuraci\u00f3n de Istio para el frontend
El chart `frontend` crea dos recursos de Istio:
- **Gateway** (`frontend/templates/gateway.yaml`) define el host `nova.dmirandam.com` con TLS y certificado `nova-crt`.
- **VirtualService** (`frontend/templates/virtual-service.yaml`) enruta las peticiones al servicio `frontend` en el puerto 80.

Aseg\u00farese de que el DNS del dominio apunte al gateway de Istio.

## 7. (Opcional) Pol\u00edtica de red para Cloudflared
Si se desea limitar el acceso al pod `cloudflared` solo a las IPs de Cloudflare, aplicar `network-policy.yaml`:
   ```bash
   kubectl apply -f network-policy.yaml
   ```

---

Con estos pasos se desplegar\u00e1n RabbitMQ, Kyverno, los microservicios y la configuraci\u00f3n de Istio para el frontend. Ajuste las variables y credenciales en los `values.yaml` de los charts seg\u00fan su entorno antes de ejecutar los comandos de Helm.
