# FleetCheck – Build Systems Lab (versão Gradle)

This project is intentionally incomplete. Follow the worksheet in the order given.

Expected final application output:

```
FleetCheck 1.0
Vehicles loaded: 4
Vehicles requiring service: 2
Average mileage: 37000 km
```

Do not copy the solution POM. The objective is to observe how each build change alters the result.

---

## Evidências – Parte Gradle

**Evidence 8.1** – Erro do `gradle clean build` sem Jackson:

```
App.java:3: error: package com.fasterxml.jackson.core.type does not exist
import com.fasterxml.jackson.core.type.TypeReference;
```

Falta a dependência `com.fasterxml.jackson.core:jackson-databind`.

**Evidence 8.2 – Mudar de ferramenta mudou as dependências?**
Não. Em `gradle dependencies --configuration runtimeClasspath`, `jackson-databind` é a dependência direta e `jackson-core` e `jackson-annotations` aparecem como transitivas — exatamente o mesmo grafo que o `mvn dependency:tree`. Só mudou a forma de declarar e de mostrar as dependências.

```
runtimeClasspath
\--- com.fasterxml.jackson.core:jackson-databind:2.22.2
     +--- com.fasterxml.jackson.core:jackson-annotations:2.22
     +--- com.fasterxml.jackson.core:jackson-core:2.22.2
     \--- com.fasterxml.jackson:jackson-bom:2.22.2 (*)
```

**Evidence 8.3 – O que mudou no JAR?**
Antes, o JAR só tinha as classes do FleetCheck e não tinha `Main-Class`, por isso `java -jar` falhava com `no main manifest attribute, in build/libs/fleetcheck-1.0.0.jar`. Depois, o manifest passou a indicar `pt.upt.fleetcheck.App` e as classes do Jackson foram extraídas do `runtimeClasspath` para dentro do JAR (fat JAR), que corre só com `java -jar` — equivalente ao `-all.jar` do Shade no Maven.

**Pergunta 8.4 – Que pressuposto do ambiente o Gradle Wrapper removeu?**
O pressuposto de que cada máquina tem o Gradle instalado, no PATH e numa versão compatível. A versão fica fixada em `gradle/wrapper/gradle-wrapper.properties` e é descarregada automaticamente, por isso todos (incluindo o CI) usam a mesma versão.

**Evidence 8.5 – GitHub Actions (Gradle)**
_(URL do run a preencher)_

**Evidence 8.6 – Porque é que o SBOM do Gradle tem dependências que não escrevi?**
Pela mesma razão que no Maven: o plugin CycloneDX percorre o grafo de dependências resolvido pelo Gradle (`runtimeClasspath`), incluindo as transitivas `jackson-core` e `jackson-annotations`, e não só a `jackson-databind` declarada no `build.gradle`.

**8.7 – Comparação Maven vs Gradle**

| Tarefa | Maven | Gradle |
|---|---|---|
| Configuração do build | `pom.xml` | `build.gradle` |
| Clean build | `./mvnw clean verify` | `./gradlew clean build` |
| Adicionar dependência | `<dependency>...</dependency>` | `implementation 'group:artifact:version'` |
| Inspecionar dependências | `mvn dependency:tree` | `gradle dependencies` |
| Wrapper | `mvnw` / `mvnw.cmd` | `gradlew` / `gradlew.bat` |
| Output do build | `target/` | `build/` |
| Localização do JAR | `target/` | `build/libs/` |
| SBOM | CycloneDX Maven plugin | CycloneDX Gradle plugin |

**Pergunta final – O que mudou: o software ou o processo de build?**
O software não mudou: o código-fonte, os recursos, as dependências e o output da aplicação são exatamente os mesmos. Mudou apenas o processo de build — a linguagem de configuração (XML declarativo vs DSL Groovy), os comandos, as pastas de output e os plugins usados para empacotar e gerar o SBOM. A qualidade do produto entregue depende de o processo de build ser reprodutível e verificável, não da ferramenta escolhida.
