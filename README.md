'''
import 'package:flutter/material.dart';
import 'dart:io';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'File Explorer',
      theme: ThemeData(
        primarySwatch: Colors.blue,
        useMaterial3: true,
      ),
      home: const FileExplorerPage(),
    );
  }
}

class FileExplorerPage extends StatefulWidget {
  const FileExplorerPage({Key? key}) : super(key: key);

  @override
  State<FileExplorerPage> createState() => _FileExplorerPageState();
}

class _FileExplorerPageState extends State<FileExplorerPage> {
  final TextEditingController _pathController = TextEditingController();
  String _hierarchyText = '';
  bool _isLoading = false;

  @override
  void initState() {
    super.initState();
    // Default path for Windows
    _pathController.text = 'C:\\Users';
  }

  void _exploreDirectory() async {
    setState(() {
      _isLoading = true;
      _hierarchyText = '';
    });

    try {
      final dir = Directory(_pathController.text);
      if (!await dir.exists()) {
        setState(() {
          _hierarchyText = 'Error: Directory does not exist!';
          _isLoading = false;
        });
        return;
      }

      final StringBuffer buffer = StringBuffer();
      await _buildHierarchy(dir, buffer, 0);

      setState(() {
        _hierarchyText = buffer.toString();
        _isLoading = false;
      });
    } catch (e) {
      setState(() {
        _hierarchyText = 'Error: ${e.toString()}';
        _isLoading = false;
      });
    }
  }

  Future<void> _buildHierarchy(Directory dir, StringBuffer buffer, int level) async {
    final indent = '  ' * level;

    try {
      final entities = await dir.list().toList();

      // Sort: directories first, then files
      entities.sort((a, b) {
        final aIsDir = a is Directory;
        final bIsDir = b is Directory;
        if (aIsDir && !bIsDir) return -1;
        if (!aIsDir && bIsDir) return 1;
        return a.path.toLowerCase().compareTo(b.path.toLowerCase());
      });

      for (var entity in entities) {
        final name = entity.path.split(Platform.pathSeparator).last;

        if (entity is Directory) {
          buffer.writeln('$indent📁 $name');
          try {
            await _buildHierarchy(entity, buffer, level + 1);
          } catch (e) {
            buffer.writeln('$indent  [Access Denied]');
          }
        } else if (entity is File) {
          buffer.writeln('$indent📄 $name');
        }
      }
    } catch (e) {
      buffer.writeln('$indent[Error reading directory: $e]');
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('File Explorer - Hierarchical View'),
        elevation: 2,
      ),
      body: Column(
        children: [
          Padding(
            padding: const EdgeInsets.all(16.0),
            child: Row(
              children: [
                Expanded(
                  child: TextField(
                    controller: _pathController,
                    decoration: const InputDecoration(
                      labelText: 'Directory Path',
                      hintText: 'C:\\Users\\YourName\\Documents',
                      border: OutlineInputBorder(),
                      prefixIcon: Icon(Icons.folder),
                    ),
                  ),
                ),
                const SizedBox(width: 16),
                ElevatedButton.icon(
                  onPressed: _isLoading ? null : _exploreDirectory,
                  icon: const Icon(Icons.search),
                  label: const Text('Explore'),
                  style: ElevatedButton.styleFrom(
                    padding: const EdgeInsets.symmetric(
                      horizontal: 24,
                      vertical: 16,
                    ),
                  ),
                ),
              ],
            ),
          ),
          const Divider(),
          Expanded(
            child: _isLoading
                ? const Center(child: CircularProgressIndicator())
                : _hierarchyText.isEmpty
                ? const Center(
              child: Text(
                'Enter a directory path and click Explore',
                style: TextStyle(fontSize: 16, color: Colors.grey),
              ),
            )
                : SingleChildScrollView(
              padding: const EdgeInsets.all(16),
              child: SelectableText(
                _hierarchyText,
                style: const TextStyle(
                  fontFamily: 'Courier',
                  fontSize: 14,
                ),
              ),
            ),
          ),
        ],
      ),
    );
  }

  @override
  void dispose() {
    _pathController.dispose();
    super.dispose();
  }
}
'''
