

  Configuración y operaciones

  
# Semántica de credenciales de autenticación

Este documento define la semántica canónica de elegibilidad y resolución de credenciales utilizada en:

-   `resolveAuthProfileOrder`
-   `resolveApiKeyForProfile`
-   `models status --probe`
-   `doctor-auth`

El objetivo es mantener alineado el comportamiento en tiempo de selección y en tiempo de ejecución.

## Códigos de Razón Estables

-   `ok`
-   `missing_credential`
-   `invalid_expires`
-   `expired`
-   `unresolved_ref`

## Credenciales de Token

Las credenciales de token (`type: "token"`) admiten `token` en línea y/o `tokenRef`.

### Reglas de elegibilidad

1.  Un perfil de token no es elegible cuando tanto `token` como `tokenRef` están ausentes.
2.  `expires` es opcional.
3.  Si `expires` está presente, debe ser un número finito mayor que `0`.
4.  Si `expires` es inválido (`NaN`, `0`, negativo, no finito o tipo incorrecto), el perfil no es elegible con `invalid_expires`.
5.  Si `expires` está en el pasado, el perfil no es elegible con `expired`.
6.  `tokenRef` no omite la validación de `expires`.

### Reglas de resolución

1.  La semántica del resolvedor coincide con la semántica de elegibilidad para `expires`.
2.  Para perfiles elegibles, el material del token puede resolverse a partir del valor en línea o de `tokenRef`.
3.  Las referencias irresolubles producen `unresolved_ref` en la salida de `models status --probe`.

## Mensajería Compatible con Versiones Anteriores

Para compatibilidad con scripts, los errores de sondeo mantienen esta primera línea sin cambios: `Auth profile credentials are missing or expired.` Los detalles amigables para humanos y los códigos de razón estables pueden añadirse en líneas posteriores.

[Autenticación](./gateway/authentication.md)[Gestión de Secretos](./gateway/secrets.md)

---