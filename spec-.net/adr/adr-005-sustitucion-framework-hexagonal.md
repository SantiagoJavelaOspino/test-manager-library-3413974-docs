# ADR-005: Sustitución de Framework por Arquitectura Hexagonal en PHP Puro

## Estado
Aceptado (Decisión Técnica y Trade-Off Asumido).

## Contexto
El enunciado general de la prueba técnica plantea como stack de referencia backend "PHP con Laravel". Sin embargo, durante el aprovisionamiento inicial y auditoría de paquetes en Composer, surgieron los siguientes condicionantes:
1. Advertencias de seguridad críticas y bloqueos por vulnerabilidades conocidas (CVEs) en componentes y paquetes vinculados al ecosistema del framework.
2. La **Constitución de SDD (Artículos I, II y III)** exige formalmente que el **Dominio** (`app/Domain`) no contenga dependencias externas ni acoplamientos a frameworks, y que la **Aplicación** (`app/Application`) desacople puertos y casos de uso mediante responsabilidad única (SRP).

## Decisión
Se decide implementar el backend utilizando **PHP 8.2 nativo con Arquitectura Hexagonal / Cebolla (Onion Architecture)**, sin depender de `laravel/framework`:
- **Dominio Puro (`app/Domain`):** Modelos e invariantes de negocio (`Book`, `Loan`, `Member`, etc.) y Value Objects inmutables (`Isbn`, `Quantity`, `DateRange`, `Role`, `Username`, `Uuid`).
- **Aplicación (`app/Application`):** Casos de uso desacoplados a través de interfaces (puertos) para repositorios, reloj, hasher y token issuer.
- **Infraestructura (`app/Infrastructure`):** Adaptadores específicos de PostgreSQL (esquema `library`), emisión nativa de JWT HMAC-SHA256 y Bcrypt hasher.
- **Enrutador HTTP (`app/Http`):** Dispatcher REST liviano bajo `/api/v1` con contrato unificado de error `{code, message, details}`.

## Alternativas Consideradas y Descartadas
1. **Ignorar advertencias de seguridad de Composer (`audit.ignore`):**
   - *Descartada:* En un entorno de desarrollo seguro, forzar la instalación de dependencias con CVEs activos vulnera las buenas prácticas de DevSecOps.
2. **Usar Laravel aislando el dominio:**
   - *Descartada:* Aunque técnicamente factible, introducir el peso de un framework completo para luego desacoplar Eloquent y el contenedor introduce complejidad accidental innecesaria para un reto de tiempo limitado donde "el Spec es el producto, no el framework".

## Consecuencias y Trade-Offs

### Positivas:
- **Cumplimiento estricto de la Constitución:** 100% de aislamiento del dominio (verificado por la prueba de arquitectura en `tests/run_tests.php`).
- **Cero dependencias externas vulnerables:** El proyecto no requiere paquetes de terceros con CVEs y se ejecuta limpiamente con la biblioteca estándar de PHP.
- **Rendimiento y portabilidad:** Arranque instantáneo, huella de memoria mínima en Docker y ejecución sin overhead de inicialización de frameworks pesados.

### Negativas / Costos Asumidos:
- **Esfuerzo de implementación:** Se debió escribir un enrutador HTTP nativo y un emisor JWT manual con HMAC-SHA256 en lugar de apoyarse en paquetes preexistentes.
- **Desviación del stack nominal del enunciado:** Se sustituyó el framework Laravel por PHP nativo, priorizando la pureza arquitectónica y la seguridad del entorno sobre la convención del framework.
