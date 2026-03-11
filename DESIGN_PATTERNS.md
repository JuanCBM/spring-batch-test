# Patrones de Diseño en Arquitectura Spring Boot

## 📋 Descripción General

Este documento describe los **15 patrones de diseño** más relevantes para construir aplicaciones empresariales escalables con **Java 17 y Spring Framework**. Cada patrón se explica de forma genérica con ejemplos de código reutilizable.

---

## 1. Patrones Arquitectónicos

### 1.1 Arquitectura por Capas

**Descripción**: Separación horizontal en tres capas independientes

```
Domain Layer (Lógica de Negocio)
    ↓
Infrastructure Layer (Implementación Técnica)
    ↓
Application Layer (Orquestación)
```

**Cuándo usar**: Siempre en aplicaciones empresariales  
**Ventajas**: Mantenibilidad, testabilidad, escalabilidad  
**Desventajas**: Complejidad inicial, más clases

**Referencia**: https://martinfowler.com/bliki/LayeredArchitecture.html

---

### 1.2 Patrón Facade

**Objetivo**: Proporcionar una interfaz simplificada a un subsistema complejo

```java
@Component
@RequiredArgsConstructor
public class OperationFacade {
    private final Service1 service1;
    private final Service2 service2;
    private final Service3 service3;
    
    public void complexOperation(String input) {
        service1.prepare(input);
        service2.process();
        service3.finalize();
    }
}
```

**Cuándo usar**: Múltiples servicios necesitan coordinación  
**Ventajas**: API simple, oculta complejidad

**Referencia**: https://refactoring.guru/design-patterns/facade

---

### 1.3 Patrón Pipeline (Chain of Responsibility)

**Objetivo**: Ejecutar una serie de pasos en secuencia

```java
@Component
public class PipelineExecutor {
    public void execute(List<Step> steps, Context context) {
        for (Step step : steps) {
            step.execute(context);
        }
    }
}

@FunctionalInterface
public interface Step {
    void execute(Context context);
}
```

**Cuándo usar**: Procesamiento secuencial con pasos intercambiables  
**Ventajas**: Flexible, separación de responsabilidades

**Referencia**: https://refactoring.guru/design-patterns/chain-of-responsibility

---

## 2. Patrones de Creación

### 2.1 Patrón Strategy

**Objetivo**: Encapsular algoritmos intercambiables

```java
public interface ConversionStrategy {
    String getName();
    void convert(Context context);
}

@Component
public class StrategyRegistry {
    private final Map<String, ConversionStrategy> strategies = new HashMap<>();
    
    public StrategyRegistry(List<ConversionStrategy> allStrategies) {
        allStrategies.forEach(s -> 
            strategies.put(s.getName().toUpperCase(), s)
        );
    }
    
    public ConversionStrategy getStrategy(String name) {
        return Optional.ofNullable(strategies.get(name.toUpperCase()))
            .orElseThrow(() -> new IllegalArgumentException(
                "Estrategia no encontrada: " + name
            ));
    }
}
```

**Cuándo usar**: Múltiples algoritmos seleccionables en tiempo de ejecución  
**Ventajas**: Fácil agregar nuevas estrategias, auto-registro automático

**Referencia**: https://refactoring.guru/design-patterns/strategy

---

### 2.2 Patrón Command

**Objetivo**: Encapsular una solicitud como objeto

```java
public interface Command {
    void execute(Context context);
}

@Component
public class SpecificCommand implements Command {
    private final Step1 step1;
    private final Step2 step2;
    
    public SpecificCommand(Step1 s1, Step2 s2) {
        this.step1 = s1;
        this.step2 = s2;
    }
    
    @Override
    public void execute(Context context) {
        step1.execute(context);
        step2.execute(context);
    }
}
```

**Cuándo usar**: Encapsular operaciones complejas para orquestación  
**Ventajas**: Desacopla solicitante de ejecutor

**Referencia**: https://refactoring.guru/design-patterns/command

---

### 2.3 Patrón Builder

**Objetivo**: Construir objetos complejos paso a paso

```java
@Data
@Builder
public class Configuration {
    private String name;
    private Integer timeout = 30;
    private List<String> endpoints;
}

Configuration config = Configuration.builder()
    .name("MyConfig")
    .timeout(60)
    .endpoints(List.of("http://api1", "http://api2"))
    .build();
```

**Cuándo usar**: Objetos con muchos parámetros opcionales  
**Ventajas**: API fluida, readabilidad

**Referencia**: https://projectlombok.org/features/Builder

---

## 3. Patrones de Comportamiento

### 3.1 Patrón Template Method

**Objetivo**: Definir esqueleto de algoritmo en clase base

```java
@FunctionalInterface
public interface Step {
    void execute(Context context);
}

@Component
public class ValidationStep implements Step {
    @Override
    public void execute(Context context) {
        if (!isValid(context.getData())) {
            throw new IllegalArgumentException("Datos inválidos");
        }
    }
    
    private boolean isValid(Object data) {
        return data != null;
    }
}
```

**Cuándo usar**: Pasos con interfaz común pero lógica diferente  
**Ventajas**: Código consistente, fácil agregar steps

**Referencia**: https://refactoring.guru/design-patterns/template-method

---

### 3.2 Dependency Injection (Spring)

**Objetivo**: Gestionar dependencias automáticamente

```java
@Component
@RequiredArgsConstructor
public class Service {
    private final Repository repository;
    private final Cache cache;
    
    public void operation() {
        repository.save(cache.get());
    }
}
```

**Cuándo usar**: Siempre con Spring  
**Ventajas**: Bajo acoplamiento, facilita testing

**Referencia**: https://docs.spring.io/spring-framework/docs/current/reference/html/core.html

---

## 4. Patrones Estructurales

### 4.1 Data Transfer Object (Record)

**Objetivo**: Transferir datos de forma inmutable

```java
public record UserDTO(
    @NotNull String id,
    @NotNull String name,
    @Email String email,
    Integer age
) {
    public UserDTO {
        if (id.isBlank()) {
            throw new IllegalArgumentException("ID requerido");
        }
    }
}

UserDTO user = new UserDTO("123", "Juan", "juan@example.com", 30);
```

**Cuándo usar**: Transferir datos entre capas  
**Ventajas**: Inmutabilidad, validación automática, código limpio

**Referencia**: https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Record.html

---

### 4.2 Patrón Generic Reader

**Objetivo**: Leer tipos diferentes de forma polimórfica

```java
public interface DataReader<T> {
    T read(Map<String, Object> source);
    Class<?> getDataType();
}

@Component
public class UserDataReader implements DataReader<UserDTO> {
    @Override
    public UserDTO read(Map<String, Object> source) {
        return new UserDTO(
            (String) source.get("id"),
            (String) source.get("name"),
            (String) source.get("email"),
            (Integer) source.get("age")
        );
    }
    
    @Override
    public Class<?> getDataType() {
        return UserDTO.class;
    }
}
```

**Cuándo usar**: Múltiples tipos de datos con interfaz común  
**Ventajas**: Extensibilidad, type-safety

**Referencia**: https://refactoring.guru/design-patterns/adapter

---

### 4.3 Patrón Registry

**Objetivo**: Gestionar colección dinámica de objetos

```java
@Component
public class OperationRegistry {
    private final Map<String, Operation> operations = new HashMap<>();
    
    public OperationRegistry(List<Operation> allOperations) {
        allOperations.forEach(op -> 
            operations.put(op.getId().toUpperCase(), op)
        );
    }
    
    public Operation getOperation(String id) {
        return operations.get(id.toUpperCase());
    }
}
```

**Cuándo usar**: Gestionar conjuntos dinámicos de objetos  
**Ventajas**: Auto-registro, fácil búsqueda

---

## 5. Patrones de Configuración

### 5.1 Spring Profiles

**Objetivo**: Ejecución diferenciada por entorno

```java
@Component
@Profile("development")
public class DevelopmentRunner implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        // Modo desarrollo
    }
}

@Component
@Profile("production")
public class ProductionRunner implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        // Modo producción
    }
}

// Ejecutar: mvn spring-boot:run -Dspring-boot.run.profiles=production
```

**Cuándo usar**: Diferentes configuraciones por entorno  
**Ventajas**: Un único código, múltiples configuraciones

**Referencia**: https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.profiles

---

### 5.2 Configuration Properties

**Objetivo**: Cargar propiedades YAML tipadas

```java
@Data
@Component
@ConfigurationProperties(prefix = "app")
public class ApplicationConfig {
    private String appName;
    private Integer timeout = 30;
    private Database database = new Database();
    
    @Data
    public static class Database {
        private String host = "localhost";
        private Integer port = 5432;
        private String name;
    }
}

// application.yml
app:
  appName: MyApplication
  timeout: 60
  database:
    host: db.internal
    port: 5432
    name: mydb
```

**Cuándo usar**: Configuración externalizada  
**Ventajas**: Type-safe, validable, documenta propiedades

**Referencia**: https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config

---

## 6. Patrones Avanzados

### 6.1 Execution Context

**Objetivo**: Compartir estado entre pasos de un pipeline

```java
@Data
public class ExecutionContext {
    private String inputPath;
    private String outputPath;
    private Map<String, Object> metadata = new HashMap<>();
    
    public void setMetadata(String key, Object value) {
        metadata.put(key, value);
    }
    
    public Object getMetadata(String key) {
        return metadata.get(key);
    }
}

@Component
public class Step1 implements Step {
    @Override
    public void execute(ExecutionContext context) {
        context.setInputPath("/input/file.txt");
        context.setMetadata("result", "processed");
    }
}

@Component
public class Step2 implements Step {
    @Override
    public void execute(ExecutionContext context) {
        Object result = context.getMetadata("result");
        log.info("Resultado anterior: {}", result);
    }
}
```

**Cuándo usar**: Pasos que necesitan compartir estado  
**Ventajas**: Evita parámetros en exceso, manejo centralizado

---

## 📊 Tabla Comparativa

| Patrón | Propósito | Complejidad | Cuándo usar |
|--------|-----------|-------------|------------|
| **Layered Arch** | Organización | Media | Siempre |
| **Facade** | Simplificar API | Baja | Múltiples servicios |
| **Pipeline** | Secuencia de pasos | Media | Procesamiento multietapa |
| **Strategy** | Algoritmos intercambiables | Baja | Múltiples opciones |
| **Command** | Encapsular operación | Media | Operaciones complejas |
| **Builder** | Construcción de objetos | Baja | Objetos complejos |
| **Template Method** | Plantilla de algoritmo | Baja | Pasos similares |
| **Dependency Injection** | Gestión de dependencias | Media | Siempre en Spring |
| **DTO/Record** | Transferencia de datos | Baja | Entre capas |
| **Generic Reader** | Lectura polimórfica | Media | Múltiples tipos |
| **Registry** | Colección dinámica | Baja | Auto-registro |
| **Profiles** | Configuración por entorno | Baja | Desarrollo vs Producción |
| **Config Properties** | Propiedades externas | Media | Externalizar configuración |
| **Execution Context** | Estado compartido | Baja | Pipelines complejos |

---

## 🎯 Guía de Selección

| Necesidad | Patrón |
|-----------|--------|
| Múltiples algoritmos intercambiables | **Strategy** |
| Ejecutar pasos secuenciales | **Pipeline + Steps** |
| Simplificar interfaz compleja | **Facade** |
| Encapsular operación compleja | **Command** |
| Objeto con muchos parámetros | **Builder** |
| Comportamientos diferentes por pasos | **Template Method** |
| Gestionar dependencias | **Dependency Injection** |
| Transferir datos entre capas | **DTO/Record** |
| Leer múltiples tipos de datos | **Generic Reader + Registry** |
| Diferente configuración por entorno | **Profiles** |
| Propiedades externas tipadas | **Configuration Properties** |
| Compartir estado entre pasos | **Execution Context** |

---

## 📚 Referencias Externas

- **Refactoring.guru**: https://refactoring.guru/design-patterns
- **Martin Fowler**: https://martinfowler.com/
- **Spring Framework**: https://docs.spring.io/spring-framework/
- **Spring Boot**: https://docs.spring.io/spring-boot/
- **Lombok**: https://projectlombok.org/
- **Java 17 Docs**: https://docs.oracle.com/en/java/javase/17/

---

**Versión**: 1.0  
**Actualizado**: 2026-03-11  
**Audiencia**: Equipos de Desarrollo con Spring Boot

