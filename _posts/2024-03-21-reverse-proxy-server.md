---
layout: post
title: Alojando un sitio web casero con Cloudflare
date: 2024-03-21 13:48:00
description: 
tags: cloudflare reverse-proxy
categories: 
toc:
    sidebar: left
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
Un servidor proxy inverso es aquel que actúa en nombre uno o más servidores web, interceptando las peticiones de los clientes para luego enviar y/o recibir peticiones hacia/desde el servidor de origen, respectivamente. Por lo tanto el cliente nunca interactúa directamente con el servidor, sino que siempre hay un intermediario al cual se le conoce como *"Man-in-the-middle"* [^1^][1] (MitM). Aunque la seguridad no fue mi principal preocupación al considerar dicha tecnología en mi proyecto, creo que vale la pena resaltar que esto añade una "capa de protección"* al evitar la comunicación directa entre cliente-servidor.

La verdadera razón por la que considero esta tecnología como la solución ideal a mi proyecto es porque, al igual que a muchos de nosotros, mi proveedor de internet aplica CGNAT. CGNAT es una variante de NAT utilizada por los proveedores de internet para que múltiples usuarios de su servicio compartan una misma IP pública. Si este no fuese mi caso probablemente no tendría justificación para elaborar este post.

[^1^][1]: Sólo si tienes la absoluta certeza que este middleman es de confiar.
