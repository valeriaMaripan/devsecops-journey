# Martes – Permisos, privilegios y escalación de privilegios

## Resumen
Se plantaron dos malas configuraciones en el servidor `target` (Ubuntu),
se detectaron mediante auditoría y se demostró la escalada a root.
Ambas fueron corregidas.

## Hallazgo 1 – Binario SUID inseguro
- **Qué:** copia de `find` en `/usr/local/bin/finder` con bit SUID de root (`-rwsr-xr-x`).
- **Cómo se detecta:** `find / -perm -4000 -type f 2>/dev/null`
- **Cómo se explota:** `finder . -exec /bin/sh \;` abre una shell como root (ref. GTFOBins).
- **Impacto:** escalada de privilegios de usuario normal a root.
- **Corrección:** eliminar el binario (`sudo rm /usr/local/bin/finder`) o quitar el SUID.

## Hallazgo 2 – Regla sudo demasiado permisiva
- **Qué:** regla en `/etc/sudoers.d/99-lab` que permite `vim` como root sin contraseña (NOPASSWD).
- **Cómo se detecta:** `sudo -l`
- **Cómo se explota:** `sudo vim -c ':!/bin/sh'` → shell root sin pedir contraseña. Confirmado con `whoami` = root.
- **Impacto:** escalada directa a root.
- **Corrección:** eliminar la regla (`sudo rm /etc/sudoers.d/99-lab`).

## Lección aprendida
Los binarios SUID y las reglas sudo son vectores clásicos de escalada de privilegios.
Hay que auditarlos periódicamente y aplicar el principio de mínimo privilegio.
