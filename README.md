# Amazon WorkSpaces: Automating Lifecycle Management & Cost Optimizer
![image](https://github.com/cbecerrae/amazon-workspaces-project/assets/143916645/b63bf55b-ae8f-4cbb-9b96-fe32897c29e8)

## Introducción
Gestionar entornos de escritorios virtuales de manera eficiente es crucial para las empresas modernas. Amazon WorkSpaces proporciona una solución segura y rentable para ofrecer escritorios virtuales, permitiendo a los usuarios acceder a sus espacios de trabajo desde cualquier lugar. Este proyecto se centra en automatizar la gestión del ciclo de vida de los WorkSpaces y optimizar sus costos, asegurando el uso eficiente de los recursos y reduciendo la carga administrativa.

### ¿Qué es Amazon WorkSpaces?
Amazon WorkSpaces es una solución completamente gestionada de Escritorio como Servicio (DaaS) que proporciona a los usuarios un escritorio virtual seguro en la nube. Permite a las empresas aprovisionar escritorios con Windows o Linux rápidamente y escalarlos según sus necesidades. Con Amazon WorkSpaces, los usuarios pueden acceder a sus entornos de escritorio desde diversos dispositivos, garantizando productividad y flexibilidad.

### Casos de Uso
Amazon WorkSpaces es ideal para una fuerza laboral remota. Permite a los empleados acceder de forma segura a sus entornos de trabajo desde cualquier lugar, utilizando diversos dispositivos. Esta solución es especialmente valiosa en el contexto actual, donde el trabajo remoto se ha convertido en una norma. Los empleados pueden llevar sus propios dispositivos y trabajar de manera eficiente sin comprometer la seguridad de los datos corporativos. Además, facilita la provisión y desprovisión rápida de escritorios para contratistas y trabajadores temporales, asegurando que solo los usuarios activos tengan acceso a los recursos necesarios. Esto no solo mejora la seguridad, sino que también optimiza los costos operativos.

### Acerca del Proyecto
Este proyecto tiene como objetivo mejorar la gestión y la eficiencia de costos de Amazon WorkSpaces a través de la automatización. La automatización y optimización de WorkSpaces no solo reduce la carga administrativa, sino que también garantiza que los recursos se utilicen de manera eficiente, contribuyendo a una reducción significativa de costos. Además, al integrar visualizaciones detalladas de métricas de uso mediante Amazon QuickSight, se proporcionan insights valiosos que permiten una mejor toma de decisiones y una gestión proactiva de los recursos.

El desarrollo del proyecto se encuentra dividido en 4 partes:
- **Infraestructura Básica**: Crear los recursos básicos iniciales que forman parte de los prerequisitos para la administración de WorkSpaces.
- **Lifecycle Management**: Automatizar el aprovisionamiento y eliminación de WorkSpaces de acuerdo a la presencia de usuarios en Active Directory.
- **Cost Optimizer**: Mensualmente ejecutar actualizaciones para convertir los WorkSpaces al Running Mode más eficiente.
- **Visualización de Reportes**: Generar un reporte cada hora con métricas de WorkSpaces para visualizar las estadísticas en QuickSight.

### Diagrama de Arquitectura
![image](https://github.com/cbecerrae/amazon-workspaces-project/assets/143916645/ca71ad1e-c477-44f1-aa48-d2b600bb6e91)

## 1. Infraestructura Básica
Antes de poder gestionar el ciclo de vida u optimizar los costos de los WorkSpaces de forma automática, se deben crear algunos recursos fijos que forman parte de lo requisitos para administrar WorkSpaces. Entre ellos se encuentra un directorio de AWS Directory Service, una instancia EC2 que actúe como Domain Controller para administrar el directorio, y de manera opcional, un File System para que los usuarios de los WorkSpaces puedan utilizar carpetas compartidas.

### 1.1. Crear el AWS Microsoft Managed AD
En **AWS Directory Service**, crea un directorio de tipo **AWS Managed Microsoft AD** y establece una contraseña para el usuario administrador del directorio.

![image](https://github.com/cbecerrae/amazon-workspaces-project/assets/143916645/059bf731-3743-4548-b5fc-05b43ce908d0)

### 1.2. Crear el Domain Controller
Despliega una instancia Amazon EC2 para administrar to AWS Microsoft Managed AD Active Directory. Antes de lanzar la instancia, cree un rol de IAM que permita unir instacias de Windows al dominio de AWS Managed Microsoft AD de acuerdo a la [Documentación de Amazon](https://docs.aws.amazon.com/es_es/directoryservice/latest/admin-guide/microsoftadbasestep3.html). 

Al momento de lanzar la instancia seleccione el rol de IAM creado, cree un Key Pair para conectarse a la instancia, y configure otros parámetros de acuerdo a la Documentación de Amazon proporcionada en el párrafo anterior.

![image](https://github.com/cbecerrae/amazon-workspaces-project/assets/143916645/a9534e90-514a-4d3b-a7a7-be0775c876c6)

Una vez lanzada la instancia, una la instancia al Security Group de domain controllers, el cual está configurado para permitir la administración del directorio desde la instancia EC2 y fue generado cuando se creó el directorio. Asimismo, cree un Security Group adicional que permita al tráfico RDP desde su IP, e incluya a la instancia en dicho grupo de seguridad.

Conéctese a la instancia utilizando RDP, inicie la aplicación **Server Manager** de Windows Server como el usuario Admin e instalé las herramientas de Active Directory en la instancia EC2 de acuerdo a la Documentación de Amazon proporcionada líneas arriba.

Ahora todo está listo para administrar a los Active Directory Users en el AWS Managed Microsoft AD. Proceda a crear una Organizational Unit (OU) llamada WorkSpaces y cree un usuario dentro de dicha OU.

![image](https://github.com/cbecerrae/amazon-workspaces-project/assets/143916645/89e624cd-64c1-44c2-9bfe-9bcb25b05ede)

### 1.3. Configurar Amazon WorkSpaces
Registre el directorio en Amazon WorkSpaces. Este registro creará un Registration Code y también creará un Security Group que controlará el tráfico de todos los WorkSpace members.

Despliegue un WorkSpace para el usuario creado, con un bundle de tipo **Standard with Windows 10 (Server 2022 based)** con un Running Mode de **AutoStop**. El WorkSpace tardará un momento en estar disponible (Available).

![image](https://github.com/cbecerrae/amazon-workspaces-project/assets/143916645/6225b979-ace5-4ef8-8f8d-63dc8a7eebdf)

Descargue el cliente de Amazon WorkSpaces y conéctese al WorkSpace utilizando el Registration code y las credenciales del Active Directory User creado.

![image](https://github.com/cbecerrae/amazon-workspaces-project/assets/143916645/aa55abaa-fcd9-4d1a-b188-cbd017273cd6)

Ahora ya tenemos un escritorio virtual de Amazon WorkSpaces disponible para un usuario, desde donde dicho usuario podrá realizar sus tareas. Sin embargo, deberíamos de tener algún tipo de almacenamiento compartido para que los usuarios de WorkSpaces puedan compartir archivos.

![image](https://github.com/cbecerrae/amazon-workspaces-project/assets/143916645/84979072-6f1c-40c0-a2c6-101a00e0e523)

### 1.4. Crear un FSx File System 
Cree un File System de **Amazon FSx for Windows File Server** para el directorio. Una vez que el File System este disponible (Available) añada a su Network interface a un nuevo Security Group que permita el tráfico entrante desde el Security Group de los WorkSpace members.

![image](https://github.com/cbecerrae/amazon-workspaces-project/assets/143916645/c034ca41-91ec-46a3-b97f-fc4dacd0f171)

Dentro del WorkSpace, siga la instrucciones de FSx para montar el File System en el WorkSpace, cree una carpeta compartida (Shared Folder) dentro del File System y añade un acceso directo en el escritorio.

![image](https://github.com/cbecerrae/amazon-workspaces-project/assets/143916645/1d072f95-072b-4b63-b13e-2e1c280b75ab)

Ahora ya tenemos una File System con carpetas compartidas donde los usuarios de WorkSpaces podrán compartir archivos.

## 2. Lifecycle Management
Esta feature permitirá automáticamente provisionar WorkSpaces cuando se creen nuevos usuarios en Active Directory y enviar correos a estos usuarios con las instrucciones para que puedan iniciar sesión en sus WorkSpaces. Asimismo, eliminará WorkSpaces en caso haya usuarios que ya no figuren en Active Directory.

### 2.1. Crear la Maintenance Window de Systems Manager
En la plantilla de CloudFormation [lifecycle-management.yaml](https://github.com/cbecerrae/amazon-workSpaces-project/blob/main/lifecycle-management.yaml) se encuentra el código para crear una Maintenance Window de Systems Manager que ejecute una Task cada 5 minutos. Esta task ingresará a la instancia Command Host, ejecutará comandos de PowerShell que obtendrá la lista actualizada de Active Directory Users y la almacenará en un bucket de S3.

En la plantilla de CloudFormation también se asignaron los permisos y políticas necesarias de Systems Manager a la Maintenance Window y a la Task.

![image](https://github.com/cbecerrae/amazon-workspaces-project/assets/143916645/36a274a0-4110-4d28-bcfa-a3848743b737)

### 2.2. Configurar un Bucket de S3
En la plantilla de CloudFormation [lifecycle-management.yaml](https://github.com/cbecerrae/amazon-workSpaces-project/blob/main/lifecycle-management.yaml) se encuentra el código para crear un bucket de S3 donde cada 5 minutos se actualizará la lista de Active Directory Users.

En la plantilla de CloudFormation también se asignó al bucket de S3 una Bucket Policy que permita acceso de lectura y escritura a dicho bucket desde la instancia EC2 *Command Host*.

![image](https://github.com/cbecerrae/amazon-workspaces-project/assets/143916645/9c1c0ea7-54bf-466b-a7e6-bfe0a3551592)

### 2.3. Crear la Función Lambda
En la plantilla de CloudFormation [lifecycle-management.yaml](https://github.com/cbecerrae/Amazon-WorkSpaces-Project/blob/main/lifecycle-management.yaml) se encuentra el código para crear una función Lambda que se ejecutará cada vez que se actualize la lista CSV en el bucket de S3.

Esta función obtendrá la lista de WorkSpaces existente y la comparará con la lista de Active Directory Users.

En caso haya un usuario en Active Directory que no tenga un WorkSpace, la función Lambda provisionará un WorkSpace y enviará las instrucciones de inicio de sesión al usuario en un email a través de SES. Cabe mencionar que la función Lambda solo creará el WorkSpace si el Active Directory User cuenta con un email asignado.

Por otro lado, en caso exista un WorkSpace para un usuario que ya no existe en Active Directory, la función Lambda eliminará el WorkSpace.

En la plantilla de CloudFormation también se asignaron los permisos necesarios para que la función Lambda pueda generar logs en CloudWatch, crear, eliminar y describir WorkSpaces, leer objetos del bucket de S3 y enviar emails mediante SES.

![image](https://github.com/cbecerrae/amazon-workspaces-project/assets/143916645/134031cc-c69a-45a3-814e-0c09cd5bfcab)

### 2.4. Configurar Amazon Simple Email Service (SES)
En Amazon SES, crea al menos un par de Identities, un correo que será el remitente de los emails, y otro correo que será el destinataroio de los emails. Ambas Identities deberán ser verificadas.

![image](https://github.com/cbecerrae/amazon-workspaces-project/assets/143916645/0f0663db-8ff1-4ede-93f0-7c6fb31ff1e4)

La verificación previa de Identities se debe a que nos encontramos en un sandbox de SES. En un entorno de producción, bastará con que registremos nuestro dominio de correo del cual somos dueños y podremos enviar correos ilimitados sin necesidad de verificar destinatarios.

### 2.5. Verificar la Funcionalidad
Crea el stack de recursos utilizando la plantilla de CloudFormation [lifecycle-management.yaml](https://github.com/cbecerrae/Amazon-WorkSpaces-Project/blob/main/lifecycle-management.yaml), para ello deberás ingresar valores para los parámetros especificados en la sección Parameters. En este caso, yo he dejado los valores apropiados para este proyecto en los campos Default de cada Parameter.

![image](https://github.com/cbecerrae/amazon-workspaces-project/assets/143916645/25c44e01-e3b0-48d9-95d5-40b48087bb3f)

Una vez desplegada la feature mediante CloudFormation, crea una nuevo usuario de Active Directory, para demostrar como la Maintenance Windows Task generará la lista de Active Directory Users en la máquina Command Host, la subirá al Bucket de S3, la función Lambda procederá a provisionar el WorkSpace y a enviar un email al usuario con instrucciones para el inicio de sesión.

![image](https://github.com/cbecerrae/amazon-workspaces-project/assets/143916645/029717cd-0354-425e-bb93-9398b8536eca)

![image](https://github.com/cbecerrae/amazon-workspaces-project/assets/143916645/e1812f6c-5737-46c2-b7b8-da9348565b08)

![image](https://github.com/cbecerrae/amazon-workspaces-project/assets/143916645/d4d19dff-6ca8-4efe-923d-7da6c32c96a3)

![image](https://github.com/cbecerrae/amazon-workspaces-project/assets/143916645/2df1a6be-2f7c-41b9-9c43-00f76467b76c)

![image](https://github.com/cbecerrae/amazon-workspaces-project/assets/143916645/42ba7a89-a9a6-4d40-aad9-c7dfb04a332d)

![image](https://github.com/cbecerrae/amazon-workspaces-project/assets/143916645/0f2e6812-d097-4120-a770-3c4fea86064f)

## 3. Cost Optimizer
El modo de ejecución (Running Mode) **AutoStop** detiene el WorkSpace automáticamente después de una cierto tiempo de inactividad lo cual permite una facturación por horas. Mientras que el Running Mode **AlwaysOn** ejecuta el WorkSpace 24/7 sin paras y tiene una facturación fija al mes.

Para determinar cuando conviene utilizar cual Running Mode, se calculó un umbral de 80 horas a partir de las tarifas actuales de WorkSpaces, es decir, a partir de 80 horas, la facturación fija de AlwaysOn es más eficiente, y para menos de 80 horas, la facturación variable de AutoStop es más económica.

La siguiente feature evaluará mensualmente la utilización de cada WorkSpace y convertirá los WorkSpaces al Running Mode más eficiente.

### 3.1. Configurar el SNS Topic
Crea un SNS Topic donde publicarás el reporte mensual de los cambios en el Running Mode de WorkSpaces. Suscribe un correo electrónico al topic y confirma la suscripción.

![image](https://github.com/cbecerrae/amazon-workspaces-project/assets/143916645/63c72094-b7e1-4b61-95de-991de1305151)

### 3.2. Crear la Función Lambda
En la plantilla de CloudFormation [cost-optimizer.yaml](https://github.com/cbecerrae/Amazon-WorkSpaces-Project/blob/main/cost-optimizer.yaml) se encuentra el código para crear una función Lambda que se ejecutará al final de cada mes, utilizará métricas de CloudWatch para obtener la utilización de los WorkSpaces de los últimos 30 días, y dependiendo de la si la utilización de cada WorkSpace supera o no el umbral de utilización, se modificará el WorkSpace para que utilice el Running Mode más eficiente.

La función Lambda también enviará un reporte al SNS Topic especificando los WorkSpaces que se modificaron al Running Mode AutoStop o AlwaysOn, o también si es que no se modificaron Running Modes este mes.

En la plantilla de CloudFormation también se asignaron los permisos necesarios para que la función Lambda pueda generar logs en CloudWatch, obtener métricas de CloudWatch, modificar y describir WorkSpaces y publicar notificaciones hacia el SNS Topic.

![image](https://github.com/cbecerrae/amazon-workspaces-project/assets/143916645/c9e44edd-01f1-4490-be5d-ee5742ff6e91)

### 3.3. Verificar la Funcionalidad
Crea el stack de recursos utilizando la plantilla de CloudFormation [cost-optimizer.yaml](https://github.com/cbecerrae/Amazon-WorkSpaces-Project/blob/main/cost-optimizer.yaml), para ello deberás ingresar valores para los parámetros especificados en la sección Parameters. En este caso, yo he dejado los valores apropiados para este proyecto en los campos Default de cada Parameter.

![image](https://github.com/cbecerrae/amazon-workspaces-project/assets/143916645/6b5fee82-e1b6-4577-ae47-f35bca23d957)

Una vez desplegada la feature mediante CloudFormation, dado que la función Lambda está diseñada para ejecutarse recién a fin de mes, haremos un test manual, ejecutaremos la función Lambda y recibiremos un email con el reporte de que no se modificaron Running Modes, esto debido a que nuestros WorkSpaces de prueba no superan el umbral de 80 de horas de utilización al mes.

![image](https://github.com/cbecerrae/amazon-workspaces-project/assets/143916645/c19a977c-1bb5-48c1-bc77-189a15d4f8f4)

![image](https://github.com/cbecerrae/amazon-workspaces-project/assets/143916645/65130187-376e-4d30-91d4-3fb88a925ee1)

## 4. Visualización de Reportes
Si bien la feature Cost Optimizer va a optimizar los WorkSpaces al Running Mode más eficiente, sería útil poder visualizar otras métricas y estadísticas sobre los WorkSpaces existentes en un Dashboard de QuickSight.

### 4.1. Crear el Bucket de S3
En la plantilla de CloudFormation [report-generator.yaml](https://github.com/cbecerrae/Amazon-WorkSpaces-Project/blob/main/report-generator.yaml) se encuentra el código para crear un bucket de S3 donde a cada hora se actualizará un reporte CSV con métricas y estadísticas sobre los WorkSpaces existentes.

![image](https://github.com/cbecerrae/amazon-workspaces-project/assets/143916645/679a6c55-1dde-4629-9a70-62af6a1d7583)

### 4.2. Crear la Función Lambda
En la plantilla de CloudFormation [report-generator.yaml](https://github.com/cbecerrae/Amazon-WorkSpaces-Project/blob/main/report-generator.yaml) se encuentra el código para crear una función Lambda que se ejecutará cada hora, utilizará métricas de CloudWatch para obtener la utilización de los WorkSpaces de los últimos 30 días, la cantidad de WorkSpaces que se encuentran Available o Stopped, y la cantidad de WorkSpaces que se encuentran en el Running Mode AutoStop o AlwaysOn. Finalmente, publicará estás métricas en un archivo CSV en el bucket de S3.

En la plantilla de CloudFormation también se asignaron los permisos necesarios para que la función Lambda pueda generar logs en CloudWatch, obtener métricas de CloudWatch, describir WorkSpaces, leer y subir objetos al bucket de S3.

![image](https://github.com/cbecerrae/amazon-workspaces-project/assets/143916645/a12b4e56-478b-46c1-90d5-17965f204adb)

### 4.3. Configurar el Dashboard de QuickSight
En QuickSight crea un nuevo dataset utilizando S3 como fuente de datos (data source), sube el archivo manifest.json requerido e ingresa la URL donde se encontrará el reporte CSV en el bucket de S3. Asimismo, durante la prueba siguiente, deberá también de otorgar acceso a QuickSight para que pueda acceder al contenido del bucket de S3.

![image](https://github.com/cbecerrae/amazon-workspaces-project/assets/143916645/df3d26ba-f2ef-4ab6-8ba1-02e3b2c2c04e)

### 4.4. Verificar la Funcionalidad
Crea el stack de recursos utilizando la plantilla de CloudFormation [report-generator.yaml](https://github.com/cbecerrae/Amazon-WorkSpaces-Project/blob/main/report-generator.yaml), una vez desplegada la feature mediante CloudFormation, dado que la función Lambda está diseñada para ejecutarse una vez por hora, haremos un test manual.

![image](https://github.com/cbecerrae/amazon-workspaces-project/assets/143916645/3967464e-3405-466b-a586-039506c2ca49)

Ejecutaremos la función Lambda, podrás visualizar que el archivo CSV correspondiente al reporte ya se encuentra en el bucket de S3. Ingresa a QuickSight, crea un Dashboard visual con estadísticas de los datos utilizando la data extraída de S3 y publica el Dashboard.

![image](https://github.com/cbecerrae/amazon-workspaces-project/assets/143916645/ebe7219e-5744-4bdd-a878-ff955877bf21)

![image](https://github.com/cbecerrae/amazon-workspaces-project/assets/143916645/6d35aa5d-10c2-4941-a87f-5da8df626a68)

![image](https://github.com/cbecerrae/amazon-workspaces-project/assets/143916645/51801d4f-137a-4f76-885d-58c3f0675bce)

El Dashboard nos muestra información sobre los WorkSpaces existentes y sus usuarios, los cuales han tenido actividad durante 4 horas y 1 hora respectivamente, asimismo nos muestra reportes respecto al estado y Running Mode de los WorkSpaces, dicha información puede ser validada con el listado de WorkSpaces del AWS Management Console.

![image](https://github.com/cbecerrae/amazon-workspaces-project/assets/143916645/0331cc3c-62dd-4759-a237-9266f9ed5f0f)

## Conclusiones

### Amazon Elastic Compute Cloud (EC2)
EC2 permitió proporcionar la capacidad de computación necesaria para desplegar una instancia de Windows Server utilizada como controlador de dominio. Su escalabilidad y flexibilidad son ideales para diversas aplicaciones en la nube.

### AWS Directory Service
AWS Managed Microsoft AD permitió facilitar la gestión de usuarios y políticas de grupo. Su integración con otros servicios de AWS simplifica la administración de identidades y recursos en la nube.

### Amazon WorkSpaces
WorkSpaces permitió ofrecer escritorios virtuales seguros y gestionados, permitiendo el trabajo remoto y optimizando la gestión de recursos mediante la automatización del aprovisionamiento y desprovisionamiento.

### Amazon FSx for Windows File Server
FSx permitió proporcionar un sistema de archivos compartido de alto rendimiento y disponibilidad, facilitando la colaboración entre usuarios de WorkSpaces.

### AWS Systems Manager
Systems Manager permitió automatizar la sincronización de usuarios de Active Directory y la gestión del ciclo de vida de WorkSpaces, mejorando la eficiencia operativa.

### Amazon Simple Storage Service (S3)
S3 permitió almacenar listas de usuarios y reportes de métricas, ofreciendo un almacenamiento escalable y seguro para datos críticos del proyecto.

### AWS Lambda
Lambda permitió automatizar la creación, eliminación y optimización de WorkSpaces sin necesidad de gestionar servidores, proporcionando una solución escalable y eficiente.

### Amazon Simple Email Service (SES)
SES permitió enviar correos electrónicos con instrucciones de inicio de sesión, asegurando una comunicación efectiva y confiable con los usuarios de WorkSpaces.

### Amazon CloudWatch
CloudWatch permitió monitorear métricas de uso de WorkSpaces, permitiendo una optimización eficiente de recursos y proporcionando visibilidad en tiempo real.

### Amazon Simple Notification Service (SNS)
SNS permitió enviar notificaciones sobre la optimización de costos de WorkSpaces, facilitando una comunicación rápida y efectiva con los administradores.

### Amazon QuickSight
QuickSight permitió visualizar métricas y estadísticas sobre el uso de WorkSpaces, proporcionando insights valiosos para la toma de decisiones y la gestión proactiva de recursos.
