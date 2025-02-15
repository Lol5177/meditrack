import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';
import 'dart:convert';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Simple Login',
      home: HomeScreen(),
    );
  }
}

class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      color: Color(0xFFC0CDD3),
      child: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Image.asset("images/image.png"),
            SizedBox(height: 20),
            ElevatedButton(
              onPressed: () {
                Navigator.push(
                  context,
                  MaterialPageRoute(builder: (context) => LoginScreen()),
                );
              },
              child: Text("Let's Get Started",
                  style: TextStyle(fontSize: 20, color: Color(0xFF07343A))),
            ),
          ],
        ),
      ),
    );
  }
}

class LoginScreen extends StatelessWidget {
  final TextEditingController emailController = TextEditingController();
  final TextEditingController passwordController = TextEditingController();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        backgroundColor: Color(0xFF3E908B),
        title: Text('Login', style: TextStyle(color: Colors.white)),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            TextField(
              controller: emailController,
              decoration: InputDecoration(
                labelText: 'Email',
                border: OutlineInputBorder(),
              ),
            ),
            SizedBox(height: 20.0),
            TextField(
              controller: passwordController,
              decoration: InputDecoration(
                labelText: 'Password',
                border: OutlineInputBorder(),
              ),
              obscureText: true,
            ),
            SizedBox(height: 20.0),
            ElevatedButton(
              onPressed: () {
                Navigator.pushReplacement(
                  context,
                  MaterialPageRoute(builder: (context) => MainScreen()),
                );
              },
              child: Text('Login'),
            ),
          ],
        ),
      ),
    );
  }
}

class MainScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return DefaultTabController(
      length: 3,
      child: Scaffold(
        appBar: AppBar(
          automaticallyImplyLeading: true,
          title: Text('MediTrack'),
          backgroundColor: Color(0xFF3E908B),
          bottom: TabBar(
            tabs: [
              Tab(text: "Reminders", icon: Icon(Icons.alarm)),
              Tab(text: "Reports", icon: Icon(Icons.description)),
              Tab(text: "Messages", icon: Icon(Icons.sms)),
            ],
          ),
        ),
        drawer: AppDrawer(),
        body: TabBarView(
          children: [
            RemindersTab(),
            Center(child: Text("Your Reports")),
            Center(child: Text("Your Messages")),
          ],
        ),
      ),
    );
  }
}

class AppDrawer extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Drawer(
      child: ListView(
        padding: EdgeInsets.zero,
        children: [
          DrawerHeader(
            decoration: BoxDecoration(color: Color(0xFF3E908B)),
            child: Text("Menu",
                style: TextStyle(color: Colors.white, fontSize: 24)),
          ),
          ListTile(
            leading: Icon(Icons.settings),
            title: Text("Settings"),
            onTap: () {
              Navigator.push(context,
                  MaterialPageRoute(builder: (context) => SettingsPage()));
            },
          ),
          ListTile(
            leading: Icon(Icons.info),
            title: Text("About MediTrack"),
            onTap: () {
              Navigator.push(context,
                  MaterialPageRoute(builder: (context) => AboutPage()));
            },
          ),
        ],
      ),
    );
  }
}

class SettingsPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("Settings")),
      body: ListView(
        padding: EdgeInsets.all(16),
        children: [
          ListTile(
            leading: Icon(Icons.notifications),
            title: Text("Notifications"),
            subtitle: Text("Manage your reminders"),
            trailing: Switch(value: true, onChanged: (val) {}),
          ),
          ListTile(
            leading: Icon(Icons.lock),
            title: Text("Privacy & Security"),
            subtitle: Text("Manage your privacy settings"),
            onTap: () {},
          ),
          ListTile(
            leading: Icon(Icons.color_lens),
            title: Text("Theme"),
            subtitle: Text("Change app theme"),
            onTap: () {},
          ),
        ],
      ),
    );
  }
}

class AboutPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("About MediTrack")),
      body: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text("MediTrack",
                style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold)),
            SizedBox(height: 10),
            Text("Version 1.0.0",
                style: TextStyle(fontSize: 16, color: Colors.grey)),
            SizedBox(height: 20),
            Text(
                "MediTrack helps you manage your medication schedule efficiently, ensuring you never miss a dose."),
            SizedBox(height: 20),
            Text("Developed by imhotep team",
                style: TextStyle(fontWeight: FontWeight.bold)),
          ],
        ),
      ),
    );
  }
}

class RemindersTab extends StatefulWidget {
  @override
  _RemindersTabState createState() => _RemindersTabState();
}

class _RemindersTabState extends State<RemindersTab> {
  List<Map<String, String>> medicines = [];

  @override
  void initState() {
    super.initState();
    _loadData();
  }

  void _loadData() async {
    SharedPreferences prefs = await SharedPreferences.getInstance();
    String? data = prefs.getString("medicines");
    if (data != null) {
      setState(() {
        medicines = (json.decode(data) as List)
            .map((item) => Map<String, String>.from(item))
            .toList();
      });
    }
  }

  void _saveData() async {
    SharedPreferences prefs = await SharedPreferences.getInstance();
    prefs.setString("medicines", json.encode(medicines));
  }

  void _deleteMedicine(int index) {
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: Text("Confirm Delete"),
        content: Text("Are you sure you want to delete this medicine?"),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: Text("Cancel"),
          ),
          ElevatedButton(
            onPressed: () {
              setState(() {
                medicines.removeAt(index);
              });
              _saveData();
              Navigator.pop(context);
            },
            child: Text("Delete"),
            style: ElevatedButton.styleFrom(backgroundColor: Colors.red),
          ),
        ],
      ),
    );
  }

  void _showAddDialog() {
    TextEditingController nameController = TextEditingController();
    TextEditingController dosageController = TextEditingController();
    TextEditingController perDayController = TextEditingController();
    TextEditingController durationController = TextEditingController();

    TimeOfDay selectedTime = TimeOfDay.now();
    final _formKey = GlobalKey<FormState>();

    showDialog(
      context: context,
      builder: (context) {
        return StatefulBuilder(
          
          builder: (context, setDialogState) {
            return AlertDialog(
              title: Text("Add New Medicine"),
              content: Form(
                key: _formKey,
                child: SingleChildScrollView(
                  child: Column(
                    mainAxisSize: MainAxisSize.min,
                    children: [
                      _buildTextField(nameController, "Medicine Name"),

                     
                      Row(
                        children: [
                          Expanded(
                            child: ElevatedButton(
                              onPressed: () async {
                                TimeOfDay? picked = await showTimePicker(
                                  context: context,
                                  initialTime: selectedTime,
                                  builder: (context, child) {
                                    return MediaQuery(
                                      data: MediaQuery.of(context).copyWith(
                                          alwaysUse24HourFormat: false),
                                      child: child!,
                                    );
                                  },
                                );

                                if (picked != null) {
                                  setDialogState(() {
                                    selectedTime =
                                        picked;
                                  });
                                }
                              },
                              child: Text("Select Time"),
                            ),
                          ),
                          SizedBox(width: 10),
                          Text(
                            _formatTime(selectedTime),
                            style: TextStyle(
                                fontSize: 16, fontWeight: FontWeight.bold),
                          ),
                        ],
                      ),

                      _buildTextField(dosageController, "Dosage"),
                      _buildTextField(perDayController, "Dosage Per Day"),
                      _buildTextField(durationController, "Treatment Duration"),
                    ],
                  ),
                ),
              ),
              actions: [
                TextButton(
                    onPressed: () => Navigator.pop(context),
                    child: Text("Cancel")),
                ElevatedButton(
                  onPressed: () {
                    if (_formKey.currentState!.validate()) {
                      setState(() {
                        medicines.add({
                          "name": nameController.text,
                          "time": _formatTime(
                              selectedTime),
                          "dosage": dosageController.text,
                          "perDay": perDayController.text,
                          "duration": durationController.text,
                        });
                      });
                      _saveData();
                      Navigator.pop(context);
                    }
                  },
                  child: Text("Add"),
                ),
              ],
            );
          },
        );
      },
    );
  }

  String _formatTime(TimeOfDay time) {
    final hour = time.hourOfPeriod == 0 ? 12 : time.hourOfPeriod;
    final minute = time.minute.toString().padLeft(2, '0');
    final period = time.period == DayPeriod.am ? "AM" : "PM";
    return "$hour:$minute $period";
  }

  Widget _buildTextField(TextEditingController controller, String label) {
    return Padding(
      padding: const EdgeInsets.symmetric(vertical: 5),
      child: TextFormField(
        controller: controller,
        decoration: InputDecoration(
          labelText: label,
          border: OutlineInputBorder(),
          errorStyle: TextStyle(color: Colors.red),
        ),
        validator: (value) {
          if (value == null || value.isEmpty) {
            return "This field cannot be empty";
          }
          return null;
        },
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Expanded(
          child: medicines.isEmpty
              ? Center(child: Text("No medicines added yet!"))
              : ListView.builder(
                  itemCount: medicines.length,
                  itemBuilder: (context, index) {
                    var medicine = medicines[index];
                    return Card(
                      margin: EdgeInsets.all(8),
                      child: ListTile(
                        title: Text(medicine["name"]!),
                        subtitle: Column(
                          crossAxisAlignment: CrossAxisAlignment.start,
                          children: [
                            _buildRow("Time", medicine["time"]!),
                            _buildRow("Dosage", medicine["dosage"]!),
                            _buildRow("Dosage Per Day", medicine["perDay"]!),
                            _buildRow("Duration", medicine["duration"]!),
                          ],
                        ),
                        trailing: IconButton(
                          icon: Icon(Icons.delete, color: Colors.red),
                          onPressed: () => _deleteMedicine(index),
                        ),
                      ),
                    );
                  },
                ),
        ),
        FloatingActionButton(
          onPressed: _showAddDialog,
          child: Icon(Icons.add),
        ),
        SizedBox(height: 20),
      ],
    );
  }

  Widget _buildRow(String title, String value) {
    return Padding(
      padding: const EdgeInsets.symmetric(vertical: 2),
      child: Row(
        children: [
          Text("$title: ", style: TextStyle(fontWeight: FontWeight.bold)),
          Expanded(child: Text(value)),
        ],
      ),
    );
  }
}
