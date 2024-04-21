---
layout: distill
title: Alojando un sitio web casero con Cloudflare
date: 2024-03-21
description: 
tags: cloudflare reverse-proxy
categories: 
authors:
    - name: Piero Carneiro
toc:
    - name: Introducción
    - name: ¿Qué es un servidor proxy inverso?
    - name: Requisitos
    - name: Manos a la obra
      subsections:
        - name: Configurando los nameservers en el dominio
        - name: Instalando el Daemon Cloudflared en Linux
# code_diff: true
---
<!-- 
- Explicar primero que es un proxy
- Explicar el proxy inverso y por que es necesario cuando nuestro proveedor de internet nos aplica CGNAT
- Manos a la obra :
    - Requisitos previos:
        - Creacion de cuenta Cloudflare
        - Un sistema operativo que este siempre encendido
        - Haber previamente comprado un dominio
    - Configuracion de los nameservers en el dominio
    - Installacion de Cloudflared & Servicio & Configuracion
    - Configuracion del servidor (en este caso nginx)
 -->

 

## Alojando un sitio web casero con Cloudflare

### Introducción
En esta entrada explicaré con el mayor detalle posible mi experiencia y las dificultades que encontré en el camino hacia el Self-Hosting.

El concepto de Self-Hosting implica tener una mayor autonomía (y responsabilidad) en la gestión del servidor, aplicaciones y datos almacenados, algo de lo cual generalmente se encarga en mayor o menor medida un proveedor de hosting.

Primero quisiera definir algunos términos y establecer algunos requisitos para tener una idea general del proyecto. 
Cabe mencionar que los requisitos a mencionar a continuación están en función de lo que yo utilicé y tenía a la mano en aquel entonces; por ello, tales requisitos no deben ser considerados estáticos ni mucho menos la única vía al Self-Hosting.

### ¿Qué es un servidor proxy inverso?
Un servidor proxy inverso es aquel que actúa en nombre uno o más servidores web, interceptando las peticiones de los clientes para luego enviar y/o recibir peticiones hacia/desde el servidor de origen, respectivamente. Por lo tanto el cliente nunca interactúa directamente con el servidor, sino que siempre hay un intermediario al cual se le conoce como *"Man-in-the-middle"* <d-footnote>Sólo si tienes la absoluta certeza que este middleman es de confiar.</d-footnote> (MitM). Aunque la seguridad no fue mi principal preocupación al considerar dicha tecnología en mi proyecto, creo que vale la pena resaltar que esto añade una "capa de protección"* al evitar la comunicación directa entre cliente-servidor.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/reverse-proxy-server/reverse_proxy_flow.png" title="Flujo de un servidor proxy inverso" class="img-fluid rounded z-depth-1" %}
    </div> 
</div>

<div class="caption">
    Flujo de un servidor proxy inverso<d-footnote>Imagen propiedad de Cloudflare.</d-footnote>
</div>

La verdadera razón por la que considero esta tecnología como la solución ideal a mi proyecto es porque, al igual que a muchos de nosotros, mi proveedor de internet aplica CGNAT. CGNAT es una variante de NAT utilizada por los proveedores de internet para que múltiples usuarios de su servicio compartan una misma IP pública. Si este no fuese mi caso probablemente no tendría justificación para elaborar este post.

### Requisitos
Como mencioné anteriormente, voy a listar aquello que usé pero que no se limita únicamente a ello:
* Poseer un dominio de cualquier proveedor
    * Que permita cambiar los nameservers
* Raspberry Pi Zero 2W, o cualquier computadora con una distribución de Linux
    * Tener el SO ya instalado y con permisos de administrador
* Una cuenta gratiuita en [Cloudflare](https://dash.cloudflare.com/sign-up)

### Manos a la obra
#### Configurando los nameservers en el dominio
Primero ingresamos a nuestro [dashboard](https://dash.cloudflare.com/) de Cloudflare y agregamos un sitio; en este caso sería el dominio que tenemos. En mi caso sería esta misma url `pierocarneiro.xyz` y seleccionamos el plan gratuito. Si luego nos aparecen más opciones por seleccionar (como los DNS records) lo ignoramos temporalmente hasta que Cloudflare nos redirija al dashboard. En el sidepanel hacemos click en **DNS->Records**.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/reverse-proxy-server/dns-records.png" title="DNS Records" class="img-fluid rounded z-depth-1" %}
    </div> 
</div>

Como podemos observar, Cloudflare ofrece sus procios nameservers, los cuales vamos a usar para reemplazar lo que ofrece por defecto nuestro proveedor de dominio.


El paso a continuación puede ser diferente por el simple hecho que las interfaces de los proveedores de dominio varia de uno en uno, por lo que espero comprendan que es básicamente imposible cubrir todos los escenarios. Aun así, considero que tratar de explicarlo servirá como una guía. En mi caso estoy utilizando porkbun.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/reverse-proxy-server/porkbun.png" title="Dashboard de porkbun" class="img-fluid rounded z-depth-1" %}
    </div> 
</div>
<div class="caption">
    Dashboard de Porkbun<d-footnote>Otros proveedores ofrecen funcionalidades similares.</d-footnote>
</div>

Ahora vamos cambiar los nameservers. Como indica la flecha roja, hacemos click en **nameservers** y nos aparecerá un recuadro con los nameservers utilizados actualmente. Algunos proveedores utilizan sus propios nameservers (como es el caso de Porkbun). Los borramos y copiamos los nameservers de Cloudflare que ha designado cuando se agregó un nuevo dominio: `fish.ns.cloudflare.com` y `mustafa.ns.cloudflare.com`.

Por último, volvemos a nuestro dashboard en Cloudflare, específicamente regresamos a **DNS->Records** para eliminar los DNS records dentro del apartado **DNS Management for...**. En caso de haber algunos, los podemos borrar con el botón de editar bajo la columna **actions**. Hacer esto con todas las entradas listadas hasta que quede totalmente vacío.

#### Instalando el Daemon Cloudflared en Linux