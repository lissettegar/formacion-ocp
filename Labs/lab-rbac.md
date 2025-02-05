### Uso de Roles predefinidos

1. Logarse en el cluster con el usuario de cada uno, obteniendo un token en la consola de OCP:

		oc login --token=xxx --server=https://xxx:6443
		Login successful.
		...


3. Para ver la lista de todos los clusterroles disponibles:

		oc get clusterrole

		NAME
		admin
		basic-user
		cluster-admin
		...
		<output omitted>
		...

4. Use el comando 'oc describe' para ver que rules estan definidas para un role en particular:

	   oc describe clusterrole/edit

	Puede ver en el resultado anterior que, por ejemplo, los usuarios con este role pueden crear y eliminar recursos como pods, configmaps, deploymentconfigs, imagestream, routes y services, pero no pueden hacer nada con los proyectos, aparte de verlos.

	Por otro lado, si hacemos un describe del cluster role view:

	   oc describe clusterrole/view

	se puede ver que las únicas acciones permitidas en los recursos son obtener, listar y observar, lo que lo convierte en una opción perfecta si, por ejemplo, desea otorgar a un equipo de desarrollo la capacidad para ver los recursos de la aplicación en producción, pero no para modificar ninguno de ellos ni crear nuevos recursos.

5. Logarse con el primer usuario de rbac:

		oc login https://xxx:6443 -u user<x>rbac1
		Login successful.
		...



6. Crear un nuevo proyecto

		oc new-project user<x>rbac1-project

7. Logarse como el segundo usuario de rbac y comprobar si se tiene acceso al proyecto del primer usuario:

		oc login https://xxx:6443 -u user<x>rbac2

		oc project user<x>rbac1-project


8. A continuación daremos permisos al usuario 2 para que tenga privilegios de "edit" en el proyecto del usuario 1:

		oc login https://xxx:6443 -u user<x>rbac1
		oc adm policy add-role-to-user edit user<x>rbac2

9. Comprobar los rolebinding existentes en el proyecto:

		oc get rolebinding

10. Ver los detalles de todos los `rolebinding` del proyecto:

		oc describe rolebinding <nombre>

11. Logarse nuevamente como user<x>rbac2 y comprobar que ahora se tiene acceso al proyecto del user<x>rbac1:

		oc login https://xxx:6443 -u user<x>rbac2
		...


12. Comprobar que el único proyecto que puede ver el user<x>rbac2 es el `user<x>rbac1-project`:

		oc get project



### Creando Roles Personalizados

Si los roles predefinidos no son suficientes, siempre puede crear roles personalizados con las reglas específicas que necesita.

1. Creemos un role personalizado que se pueda usar en lugar del role de edición para crear y obtener pods. Logarse con el usuario de cada uno con un token:

		oc login --token=xxx --server=https://xxx:6443
		export GUID=<lgp>
		oc create clusterrole custom-role-${GUID} --verb=get,list,watch --resource=namespace,project

	Note que hemos iniciado sesión como cluster-admin para crear un `Clusterrole`.

2. Ver los detalles del nuevo `Clusterrole`:

		oc describe clusterrole custom-role-${GUID}
		Name:         custom-role-lgp
		Labels:       <none>
		Annotations:  <none>
		PolicyRule:
		  Resources                      Non-Resource URLs  Resource Names  Verbs
		  ---------                      -----------------  --------------  -----
		  namespaces                     []                 []              [get list watch]
		  projects.project.openshift.io  []                 []              [get list watch]

3. Añadir el nuevo clusterrole al usuario user<x>rbac2

		oc adm policy add-cluster-role-to-user custom-role-${GUID} user<x>rbac2

		oc get clusterrolebinding |grep $GUID

		oc describe clusterrolebinding custom-role-${GUID}


4. Comprobar cuantos proyectos puede ver ahora el user<x>rbac2:

		oc login https://xxx:6443 -u user<x>rbac2

		oc get project


5. Limpiar el entorno:

		oc login -u user<x> --server=https://xxx:6443
		oc delete clusterrolebinding custom-role-${GUID}
		oc delete project user<x>rbac1-projec		
