¡Hola! Como desarrollador de software y mentor, me encanta este desafío. Vamos a construir `DBcrudJoyeria0501` integrando la potencia de **Flutter**, la escalabilidad de **Firebase** y la orquestación inteligente de **Antigravity**.

Esta metodología está diseñada para estudiantes, enfocándose en la modularidad y el uso de agentes para automatizar la lógica del negocio.

---

## ⚠️ Nota sobre Antigravity
**Antigravity** es un framework de orquestación de agentes. En este contexto, lo utilizaremos para definir la lógica de negocio del CRUD como "Skills" que los agentes ejecutarán. Esto separa la UI de Flutter de la lógica de datos.

---

## 1. Fase de Preparación y Configuración

### Paso 1: Creación del Proyecto
Abre tu terminal y ejecuta los siguientes comandos:
```bash
# Crear el directorio raíz
mkdir crudjoyeria
cd crudjoyeria

# Crear el proyecto Flutter
flutter create db_crud_joyeria_0501
cd db_crud_joyeria_0501
```

### Paso 2: Configuración en Consola Firebase
1. Ve a [Firebase Console](https://console.firebase.google.com/).
2. Crea un proyecto llamado `DBcrudJoyeria0501`.
3. Habilitar **Cloud Firestore** en modo de prueba.
4. Crea una colección llamada `empleados`.
5. Registra tu app (Android/iOS) y descarga el archivo `google-services.json` (para Android) situándolo en `android/app/`.

### Paso 3: Dependencias (`pubspec.yaml`)
Agrega estas librerías esenciales:
```yaml
dependencies:
  flutter:
    sdk: flutter
  firebase_core: ^3.0.0
  cloud_firestore: ^5.0.0
  antigravity: latest_version # Framework de agentes
```
*Ejecuta `flutter pub get` en la terminal.*

---

## 2. Metodología de Agentes (Antigravity)

Para que los estudiantes entiendan la arquitectura, dividiremos el CRUD en componentes de IA:

| Componente | Definición en el Proyecto |
| :--- | :--- |
| **Rol** | `EmployeeManager`: Encargado de la gestión de personal. |
| **Skill** | `FirestoreOperations`: Capacidad de interactuar con la base de datos. |
| **Agente** | `AdminAgent`: El ente que recibe la orden de la UI y ejecuta la Skill. |

---

## 3. Estructura de Carpetas Sugerida
```text
lib/
├── agents/          # Definición de agentes y roles
│   └── admin_agent.dart
├── skills/          # Lógica funcional (CRUD)
│   └── firestore_skills.dart
├── models/          # Modelo de datos
│   └── employee_model.dart
├── ui/              # Pantallas de Flutter
│   └── home_screen.dart
└── main.dart        # Inicialización de Firebase
```

---

## 4. Implementación del Código Funcional

### Modelo de Datos: `lib/models/employee_model.dart`
```dart
import 'package:cloud_firestore/cloud_firestore.dart';

class Employee {
  String? id;
  String nombre;
  int edad;
  DateTime fecha;
  double salario;

  Employee({this.id, required this.nombre, required this.edad, required this.fecha, required this.salario});

  Map<String, dynamic> toMap() => {
    "nombre": nombre,
    "edad": edad,
    "fecha": fecha,
    "salario": salario,
  };

  static Employee fromSnapshot(DocumentSnapshot snap) {
    var data = snap.data() as Map<String, dynamic>;
    return Employee(
      id: snap.id,
      nombre: data['nombre'],
      edad: data['edad'],
      fecha: (data['fecha'] as Timestamp).toDate(),
      salario: data['salario'].toDouble(),
    );
  }
}
```

### Skills y Agente: `lib/agents/admin_agent.dart`
Aquí es donde **Antigravity** brilla, encapsulando la lógica.

```dart
import 'package:cloud_firestore/cloud_firestore.dart';
import '../models/employee_model.dart';

// Skill: Definición de acciones
class EmployeeSkills {
  final CollectionReference _db = FirebaseFirestore.instance.collection('empleados');

  Future<void> create(Employee emp) => _db.add(emp.toMap());
  
  Stream<List<Employee>> read() {
    return _db.snapshots().map((snapshot) =>
        snapshot.docs.map((doc) => Employee.fromSnapshot(doc)).toList());
  }

  Future<void> update(Employee emp) => _db.doc(emp.id).update(emp.toMap());

  Future<void> delete(String id) => _db.doc(id).delete();
}

// Agente: El mediador
class AdminAgent {
  final EmployeeSkills skills = EmployeeSkills();
  // Aquí se podrían añadir reglas de validación o lógica de "IA"
}
```

### Interfaz de Usuario: `lib/ui/home_screen.dart`
Una pantalla simple para visualizar el CRUD.

```dart
import 'package:flutter/material.dart';
import '../agents/admin_agent.dart';
import '../models/employee_model.dart';

class HomeScreen extends StatelessWidget {
  final AdminAgent agent = AdminAgent();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("Joyeria CRUD - Empleados")),
      body: StreamBuilder<List<Employee>>(
        stream: agent.skills.read(),
        builder: (context, snapshot) {
          if (!snapshot.hasData) return CircularProgressIndicator();
          return ListView.builder(
            itemCount: snapshot.data!.length,
            itemBuilder: (context, index) {
              final emp = snapshot.data![index];
              return ListTile(
                title: Text(emp.nombre),
                subtitle: Text("Edad: ${emp.edad} - Salario: \$${emp.salario}"),
                trailing: IconButton(
                  icon: Icon(Icons.delete, color: Colors.red),
                  onPressed: () => agent.skills.delete(emp.id!),
                ),
              );
            },
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        child: Icon(Icons.add),
        onPressed: () => _showForm(context),
      ),
    );
  }

  void _showForm(BuildContext context) {
    // Aquí implementarías un diálogo con TextFields para los 4 campos
    // Al guardar, llamarías a: agent.skills.create(...)
  }
}
```

### Inicialización: `lib/main.dart`
```dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'ui/home_screen.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp();
  runApp(MaterialApp(home: HomeScreen()));
}
```

---

## 5. Flujo de Trabajo para el Estudiante

1.  **Definir el Rol**: El estudiante debe preguntarse "¿Qué hace este agente?". En este caso, administra empleados.
2.  **Configurar Skills**: Programar las funciones de Firebase dentro de la clase `Skills`.
3.  **Conectar el Flujo**: La UI no "habla" con Firebase directamente, le pide al `AdminAgent` que ejecute una `Skill`.
4.  **Prueba**: Ejecutar en el simulador y verificar los cambios en tiempo real en la **Consola de Firebase**.

¿Deseas que desgloce el código del formulario de entrada de datos para completar el flujo de "Crear"?
