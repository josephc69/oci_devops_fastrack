# Desplegando Infraestructura como código - Oracle Resource Manager

Cuando deseamos desplegar infraestructuras más complejas, que poseen muchos recursos que dependen entre si, es necesario hacer uso de una herramienta mucho más robusta. Para este tutorial haremos uso de Terraform que nos permite precisamente esto mediante el uso de una lenguaje sencillo conocido como HCL (Hashicorp Configuration Language) para describir nuestra arquitectura.

¿Pero cómo desplegamos este código? es mala prácticamente que lo hagamos desde nuestra propia laptop por tanto es ideal un lugar centralizado que guarde un registro de los cambios y tareas ejecutadas con este código; para esto usaremos Resource Manager que es una solución de OCI que te permite precisamente centralizar los deployments de terraform y mantener un registro de cambios de ejecución.


## Resource Manager Workflow

Cuando estamos usando Resource Manager el flujo de tareas que podemos hacer esta representado en la siguiente imagen.

![rm](/img/resourcemanager/resource_manager_workflow.jpg)

## Pasos 1. Crear el stack

Lo primero es comprimir los archivos terraform con extención tf ubicados en el directorio ./iac/terraform en formato zip.

Si estas en powershell
```powershell
Compress-Archive -Path {directorio de archivos terraform}\* -DestinationPath ".\stack.zip"
```
Si estas en linux 

```shell
zip -j stack.zip ../iac/terraform/
```

Copiamos el compartment id en donde se creará el stack

```
oci iam compartment list --all #Listar todos los compartments en tu tenant
```
Ahora vamos a crear el stack, como puedes ver más abajo el comando $cid representa el OCID del compartment donde vamos a crear el stack, adicionalmente añadimos un nombre al stack y la versión de terraform que vamos a usar que en este caso es 0.12.x.

```shell
oci resource-manager stack create --compartment-id $cid --config-source stack.zip --display-name "Demo Stack" --terraform-version "0.12.x"
```

Ejecutado el comando debemos tener el siguiente output en nuestra terminal, tomemos nota del "id"; este es nuestro stack-id para futuras operaciones.

![stack create out](/img/resourcemanager/create_stack_output.jpg)

### Si nos vamos a la consola, veremos el stack creado via oci-cli 

Vamos a ver nuestro recién creado stack y además podemos hacer clic y explorar todas las opciones que podemos tomar en Resource Manager. La idea es que podamos desplegar nuestras receta terraform tal y como lo hariamos desde nuestra máquina pero desde una lugar centralizado sin la necesidad de tener un servidor de CI/CD para esto.

![new stack](/img/resourcemanager/create_stack_output_ui.jpg)

![stack console](/img/resourcemanager/stack_created_dashboard.jpg)

## Pasos 2. Terraform Plan

Vamos primero primero a ejecutar un Terraform plan via Resource manager.

   ```powershell
   oci resource-manager job create-plan-job --stack-id "ocid1.ormstack.oc1.phx.aaaaaaaa35d5mvdzdlmdabjevxuk3sb6vh3weld4nq6jcldnv5fw5fdhnvqq"
   ```

   ![plan output](/img/resourcemanager/terminal_plan_stack_output.jpg)

Como podemos observar en la imagen de arriba, vemos que el plan fue aceptado, lo que ocurre ahora es que en Resource Manager se esta ejecutando un "Terraform plan".

![plan UI out](/img/resourcemanager/plan_stack_output.jpg)

En el output en terminal, debemos anotar el "id" para luego ejecutar nuestro Terraform apply.

## Pasos 3. Terraform apply
Vamos a ejecturar terraform apply haciendo uso del terraform plan previo.

   ```shell
   oci resource-manager job create-apply-job --stack-id "ocid1.ormstack.oc1.phx.aaaaaaaa35d5mvdzdlmdabjevxuk3sb6vh3weld4nq6jcldnv5fw5fdhnvqq" --execution-plan-strategy "FROM_PLAN_JOB_ID" --execution-plan-job-id "ocid1.ormjob.oc1.phx.aaaaaaaa27tt62iwp3gixrbp2cekhzqwku62xt5w5qmlmhk6vrozlwsvvzoa"
   ```

 El "execution plan strategy" determina si se usara un plan Job previo o si primero hará plan y luego apply.

![apply output](/img/resourcemanager/terminal_appy_stack_output.jpg)

En consola mientras esta en pending veremos esto.

![pending ui](/img/resourcemanager/apply_stack_output_pending.jpg)

Nos toca esperar un poco mientras se crear nuestra Red, Cluster de Kubernetes y su nodo!!!... just chill!

Luego vamos de nuevo a la consola y veremos el resultado de nuestro apply. Ya tenemos nuestro cluster listo!

![terraform output](/img/resourcemanager/apply_stack_output_success.jpg)