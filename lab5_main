import 'package:flutter/material.dart';
import 'package:hive/hive.dart';
import 'package:hive_flutter/hive_flutter.dart';
import 'package:connectivity_plus/connectivity_plus.dart';
import 'api_service.dart';



void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Hive.initFlutter();
  await Hive.openBox('userBox'); // Відкриваємо сховище Hive
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Hive Auth App',
      theme: ThemeData(primarySwatch: Colors.blue),
      home: const AuthPage(),
    );
  }
}

class AuthPage extends StatelessWidget {
  const AuthPage({super.key});
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Авторизація')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            ElevatedButton(
              onPressed: () {
                Navigator.push(context, MaterialPageRoute(builder: (_) => const RegistrationPage()));
              },
              child: const Text('Реєстрація'),
            ),
            const SizedBox(height: 20),
            ElevatedButton(
              onPressed: () {
                Navigator.push(context, MaterialPageRoute(builder: (_) => const LoginPage()));
              },
              child: const Text('Вхід'),
            ),
          ],
        ),
      ),
    );
  }
}

class RegistrationPage extends StatefulWidget {
  const RegistrationPage({super.key});
  @override
  State<RegistrationPage> createState() => _RegistrationPageState();
}

class _RegistrationPageState extends State<RegistrationPage> {
  final _formKey = GlobalKey<FormState>();
  String _email = '';
  String _password = '';
  String _confirmPassword = '';
  String _name = '';

  Future<void> _register() async {
    if (_validateInputs()) {
      final userBox = Hive.box('userBox');
      List<dynamic> users = userBox.get('users', defaultValue: []);
      if (users.any((user) => user['email'] == _email)) {
        ScaffoldMessenger.of(context).showSnackBar(
          const SnackBar(content: Text('Користувач із такою поштою вже існує')),
        );
        return;
      }
      users.add({'email': _email, 'password': _password, 'name': _name});
      await userBox.put('users', users);
      // Перенаправлення на головну сторінку після успішної реєстрації
      Navigator.pushReplacement(
        context,
        MaterialPageRoute(builder: (_) => HomePage(email: _email)),
      );
    }
  }

  bool _validateInputs() {
    if (_formKey.currentState!.validate()) {
      if (_password != _confirmPassword) {
        ScaffoldMessenger.of(context).showSnackBar(
          const SnackBar(content: Text('Паролі не співпадають')),
        );
        return false;
      }
      return true;
    }
    return false;
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Реєстрація')),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Form(
          key: _formKey,
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              TextFormField(
                decoration: const InputDecoration(labelText: 'Ім’я'),
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Будь ласка, введіть ваше ім’я';
                  }
                  if (RegExp(r'[0-9]').hasMatch(value)) {
                    return 'Ім’я не повинно містити цифр';
                  }
                  return null;
                },
                onChanged: (value) {
                  setState(() => _name = value);
                },
              ),
              TextFormField(
                decoration: const InputDecoration(labelText: 'Електронна пошта'),
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Будь ласка, введіть вашу пошту';
                  }
                  if (!value.contains('@')) {
                    return 'Некоректний формат пошти';
                  }
                  return null;
                },
                onChanged: (value) {
                  setState(() => _email = value);
                },
              ),
              TextFormField(
                decoration: const InputDecoration(labelText: 'Пароль'),
                obscureText: true,
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Будь ласка, введіть пароль';
                  }
                  if (value.length < 6) {
                    return 'Пароль повинен бути мінімум 6 символів';
                  }
                  return null;
                },
                onChanged: (value) {
                  setState(() => _password = value);
                },
              ),
              TextFormField(
                decoration: const InputDecoration(labelText: 'Підтвердьте пароль'),
                obscureText: true,
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Будь ласка, підтвердьте пароль';
                  }
                  return null;
                },
                onChanged: (value) {
                  setState(() => _confirmPassword = value);
                },
              ),
              const SizedBox(height: 20),
              ElevatedButton(
                onPressed: _register,
                child: const Text('Зареєструватися'),
              ),
            ],
          ),
        ),
      ),
    );
  }
}

class LoginPage extends StatefulWidget {
  const LoginPage({super.key});
  @override
  State<LoginPage> createState() => _LoginPageState();
}

class _LoginPageState extends State<LoginPage> {
  final _formKey = GlobalKey<FormState>();
  String _email = '';
  String _password = '';

  Future<void> _login() async {
    final userBox = Hive.box('userBox');
    List<dynamic> users = userBox.get('users', defaultValue: []);
    final user = users.firstWhere(
      (user) => user['email'] == _email && user['password'] == _password,
      orElse: () => null,
    );
    if (user != null) {
      Navigator.pushReplacement(
        context,
        MaterialPageRoute(builder: (_) => HomePage(email: user['email'])),
      );
    } else {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(content: Text('Неправильний логін або пароль')),
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Вхід')),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Form(
          key: _formKey,
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              TextFormField(
                decoration: const InputDecoration(labelText: 'Електронна пошта'),
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Будь ласка, введіть вашу пошту';
                  }
                  if (!value.contains('@')) {
                    return 'Некоректний формат пошти';
                  }
                  return null;
                },
                onChanged: (value) {
                  setState(() => _email = value);
                },
              ),
              TextFormField(
                decoration: const InputDecoration(labelText: 'Пароль'),
                obscureText: true,
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Будь ласка, введіть пароль';
                  }
                  return null;
                },
                onChanged: (value) {
                  setState(() => _password = value);
                },
              ),
              const SizedBox(height: 20),
              ElevatedButton(
                onPressed: () {
                  if (_formKey.currentState!.validate()) {
                    _login();
                  }
                },
                child: const Text('Увійти'),
              ),
            ],
          ),
        ),
      ),
    );
  }
}

class HomePage extends StatefulWidget {
  final String email;
  const HomePage({super.key, required this.email});
  @override
  State<HomePage> createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  @override
  void initState() {
    super.initState();
    Connectivity().onConnectivityChanged.listen((ConnectivityResult result) {
      if (result == ConnectivityResult.none) {
        showDialog(
          context: context,
          builder: (_) => AlertDialog(
            title: const Text('Відсутнє з\'єднання з інтернетом!'),
            actions: [
              TextButton(
                onPressed: () => Navigator.pop(context),
                child: const Text('OK'),
              ),
            ],
          ),
        );
      }
    });
  }

  void _logout() async {
    Navigator.pushReplacement(context, MaterialPageRoute(builder: (_) => const AuthPage()));
  }

  void _openProfile() {
    Navigator.push(
      context,
      MaterialPageRoute(builder: (_) => ProfilePage(email: widget.email)),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Головна сторінка'),
        actions: [
          IconButton(
            icon: const Icon(Icons.person),
            onPressed: _openProfile,
          ),
          IconButton(
            icon: const Icon(Icons.logout),
            onPressed: _logout,
          ),
        ],
      ),
      body: const Center(
        child: Text('Ласкаво просимо на головну сторінку!'),
      ),
    );
  }
}

class ProfilePage extends StatelessWidget {
  final String email;
  const ProfilePage({super.key, required this.email});
  @override
  Widget build(BuildContext context) {
    final userBox = Hive.box('userBox');
    final users = userBox.get('users', defaultValue: []);
    final user = users.firstWhere((user) => user['email'] == email, orElse: () => null);
    if (user == null) {
      return Scaffold(
        appBar: AppBar(title: const Text('Профіль')),
        body: const Center(child: Text('Користувача не знайдено')),
      );
    }
    return Scaffold(
      appBar: AppBar(title: const Text('Профіль')),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            const Text('Дані профілю:', style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
            const SizedBox(height: 10),
            Text('Ім’я: ${user['name']}'),
            Text('Електронна пошта: ${user['email']}'),
            const SizedBox(height: 20),
            ElevatedButton(
              onPressed: () {
                Navigator.pop(context);
              },
              child: const Text('Назад до головної сторінки'),
            ),
          ],
        ),
      ),
    );
  }
}
