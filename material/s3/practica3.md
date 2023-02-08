# Práctica 3. Servicios de proceso y redes
## Requerimientos
- Una cuenta de Azure

## Principales escenarios
Esta sesión estará abordando los tres pincipales escenarios en los que un desarrollador se podría topar. No existen estos únicos tres, pero son los más comunes para las personas que apenas están iniciando con los temas de procesos y redes en Azure.

- Crear una máquina virtual de Windows
- Crear una máquina virtual de Linux
- Configurar el acceso de red

## Creación de una máquina virtual de Windows

![image](https://user-images.githubusercontent.com/50784966/217602016-59dbb3bc-d05b-4d22-98a1-0e5e417d3a7e.png)

Accede al portal de Azure y selecciona **Crear un nuevo recurso** en el apartado de Servicios de Azure.

![image](https://user-images.githubusercontent.com/50784966/217603053-f922339b-b55d-4b4e-96b0-9740bd42c647.png)

Accederá a una página como la que se muestra a continuación

![image](https://user-images.githubusercontent.com/50784966/217602503-0cf69f71-1ea0-46c1-ace6-2c8396f2618d.png)

Utilice el buscador para encontrar el recurso **Máquina Virtual**. Si lo requiere, puede usar el recurso **Máquina Virtual para cuenta gratuita**.

![image](https://user-images.githubusercontent.com/50784966/217603658-20cd7f68-64db-4c03-9e34-7b9b8b19271e.png)

Aprovisiona el recurso, dando a la opción **Crear**. Seleccione un nombre, un grupo de recursos, una región y alguna de las opciones de Sistema Operativo de Windows.

![image](https://user-images.githubusercontent.com/50784966/217608888-be084f63-97e2-4e08-9408-ec942e2ded59.png)

En la misma pantalla, seleccione una arquitectura, un tamaño, así como designar un usuario y una contraseña para acceder a la Máquina Virtual. Asi mismo, selecciona la opción de que dispones de una licencia válida de Windows. Haz clic en Siguiente.

![image](https://user-images.githubusercontent.com/50784966/217609748-7fa6174c-2063-4f24-a954-df4a2602b4b3.png)

Seleccione el disco **SSD estándar** y haga clic en Revisar y crear.

![image](https://user-images.githubusercontent.com/50784966/217610404-31d71fb6-b43a-49b3-a5b2-86064068aed0.png)

El proceso tomará algunos minutos.

### Acceder a una máquina virtual de Windows

Para acceder a la máquina virtual desde Windows, buscamos en nuestro computador **Conexión a escritorio remoto**

![image](https://user-images.githubusercontent.com/50784966/217614163-89f4a005-e764-4f0d-a2bd-7333abab41db.png)

## Creación de una máquina virtual de Linux 
Desde Cloud Shell, ejecute el siguiente comando ``az vm create`` para crear una máquina virtual Linux:
```
az vm create \
  --resource-group [resource group name] \
  --name my-vm \
  --image UbuntuLTS \
  --admin-username azureuser \
  --generate-ssh-keys
```
La máquina virtual tardará unos minutos en aparecer. Asigne el nombre ``my-vm`` a la máquina virtual. Ejecute el siguiente comando ``az vm extension set`` para configurar Nginx en la máquina virtual:
```
az vm extension set \
  --resource-group [resource group name] \
  --vm-name my-vm \
  --name customScript \
  --publisher Microsoft.Azure.Extensions \
  --version 2.1 \
  --settings '{"fileUris":["https://raw.githubusercontent.com/MicrosoftDocs/mslearn-welcome-to-azure/master/configure-nginx.sh"]}' \
  --protected-settings '{"commandToExecute": "./configure-nginx.sh"}'
```

## Confirgurar el acceso de red
En este momento, la máquina virtual de Linux no es accesible desde Internet. Creará un grupo de seguridad de red que lo cambiará y permitirá el acceso HTTP de entrada en el puerto 80.

En este laboratorio se mostrará el ejemplo utiliznado el portal de Azure. Para el laboratorio utilizando la Cloud Shell, visite el módulo de Microsoft Learn [Configuración del acceso de red](https://learn.microsoft.com/es-mx/training/modules/describe-azure-compute-networking-services/9-exercise-configure-network-access?wt.mc_id=studentamb_159817)

<!--
En este procedimiento se obtiene la dirección IP de la máquina virtual y se intenta acceder a la página principal del servidor web. Ejecute el siguiente comando az vm list-ip-addresses para obtener la dirección IP de la máquina virtual y almacenar el resultado como una variable de Bash:

```
IPADDRESS="$(az vm list-ip-addresses \
  --resource-group [resource group name] \
  --name my-vm \
  --query "[].virtualMachine.network.publicIpAddresses[*].ipAddress" \
  --output tsv)"
 ```

Ejecute el siguiente comando curl para descargar la página principal:
```
curl --connect-timeout 5 http://$IPADDRESS
```
El argumento --connect-timeout especifica que se conceden hasta cinco segundos para que se produzca la conexión. Después de cinco segundos, verá un mensaje de error que indica que se ha agotado el tiempo de espera de la conexión:

```
curl: (28) Connection timed out after 5001 milliseconds
```
Este mensaje significa que no se pudo acceder a la máquina virtual dentro del tiempo de espera.

No se pudo acceder al servidor web. Para averiguar el motivo, vamos a examinar las reglas actuales del grupo de seguridad de red. Ejecute el siguiente comando az network nsg list para que muestre los grupos de seguridad de red asociados a la máquina virtual:

```
az network nsg list \
  --resource-group [resource group name] \
  --query '[].name' \
  --output tsv
```
Verá lo siguiente:
```
my-vmNSG
```
Cada máquina virtual de Azure está asociada a, al menos, un grupo de seguridad de red. En este caso, Azure le creó un grupo de seguridad de red denominado ``my-vmNSG``.
Ejecute el siguiente comando az network nsg rule list para mostrar las reglas asociadas al grupo de seguridad de red denominado ``my-vmNSG``:

```
az network nsg rule list \
  --resource-group [resource group name] \
  --nsg-name my-vmNSG
```
Verá un bloque grande de texto en formato JSON en la salida. En el paso siguiente, ejecutará un comando similar que facilita la lectura de este resultado.

Ejecute por segunda vez el comando ``az network nsg rule list``. Esta vez, use el argumento ``--query`` para recuperar solo el nombre, la prioridad, los puertos afectados y el acceso (Permitir o Denegar) para cada regla. El argumento --output da formato a la salida como una tabla para que sea fácil de leer.

```
az network nsg rule list \
  --resource-group [sandbox resource group name] \
  --nsg-name my-vmNSG \
  --query '[].{Name:name, Priority:priority, Port:destinationPortRange, Access:access}' \
  --output table
```
Verá lo siguiente:

```
Name              Priority    Port    Access
-----------------  ----------  ------  --------
default-allow-ssh  1000        22      Allow
```
Verá la regla predeterminada ``default-allow-ssh``. Esta regla permite conexiones entrantes a través del puerto 22 (SSH). SSH (Secure Shell) es un protocolo que se usa en Linux para permitir que los administradores accedan al sistema de forma remota. La prioridad de esta entrada es 1000. Las reglas se procesan en orden de prioridad, donde los números más bajos se procesan antes que los números más altos.

De forma predeterminada, un grupo de seguridad de red de una máquina virtual de una máquina virtual Linux solo permite el acceso a la red en el puerto 22. Esto permite que los administradores accedan al sistema. También debe permitir las conexiones entrantes en el puerto 80, que permite el acceso a través de HTTP.


En este caso, creará una regla de seguridad de red que permita el acceso de entrada en el puerto 80 (HTTP).

Ejecute el siguiente comando ``az network nsg rule create`` para crear una regla denominada ``allow-http`` que permita el acceso entrante en el puerto 80:

```
az network nsg rule create \
  --resource-group [sandbox resource group name] \
  --nsg-name my-vmNSG \
  --name allow-http \
  --protocol tcp \
  --priority 100 \
  --destination-port-range 80 \
  --access Allow
```
Con fines de aprendizaje, aquí establecerá la prioridad en 100. En este caso, la prioridad no importa. Tendrá que tener en cuenta la prioridad si tuviera intervalos de puertos superpuestos.

Para comprobar la configuración, ejecute az network nsg rule list para ver la lista actualizada de reglas:

```
az network nsg rule list \
  --resource-group [resource group name] \
  --nsg-name my-vmNSG \
  --query '[].{Name:name, Priority:priority, Port:destinationPortRange, Access:access}' \
  --output table
```
Verá las dos reglas, ``default-allow-ssh`` y la nueva regla ``allow-http``:

```
Name              Priority    Port    Access
-----------------  ----------  ------  --------
default-allow-ssh  1000        22      Allow
allow-http        100        80      Allow
```

Ahora que ha configurado el acceso de red al puerto 80, vamos a intentar acceder al servidor web una segunda vez. Ejecute el mismo comando curl que ha ejecutado antes:
```
curl --connect-timeout 5 http://$IPADDRESS
```
Verá lo siguiente:

```
<html><body><h2>Welcome to Azure! My name is my-vm.</h2></body></html>
```
-->
