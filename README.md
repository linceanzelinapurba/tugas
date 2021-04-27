# tugas
Tugas flutter
class PenjualanBarang {
  PenjualanBarang({
    this.no,
    this.nama_barang,
    this.jumlah,
    this.harga,
  });

  String no;
  String nama_barang;
  String jumlah;
  int harga;
}

import 'package:explore_data/useCase/sqflite/PenjualanBarang.dart';
import 'package:path/path.dart';
import 'package:sqflite/sqflite.dart';

class MySqflite {
  static final _databaseName = "MyDatabase.db";

  static final _databaseV1 = 1;
  static final tablepenjualan = 'penjualan';

  static final columnno = 'no';
  static final columnnama_barang= 'nama barang';
  static final columnjumlah = 'jumlah';
  static final columnharga = 'harag';

  // make this a singleton class
  MySqflite._privateConstructor();

  static final MySqflite instance = MySqflite._privateConstructor();

  static Database _database;

  Future<Database> get database async {
    if (_database != null) return _database;
    _database = await _initDatabase();
    return _database;
  }

  _initDatabase() async {
    var databasesPath = await getDatabasesPath();
    String path = join(databasesPath, _databaseName);

    return await openDatabase(path, version: _databaseV1,
        onCreate: (db, version) async {
      var batch = db.batch();
      _onCreateTableMahasiswa(batch);

      await batch.commit();
    });
  }

  void _onCreateTablePenjualan(Batch batch) async {
    batch.execute('''
          CREATE TABLE $tablePenjualan (
            $columnNim TEXT PRIMARY KEY,
            $columnName TEXT,
            $columnDepartment TEXT,
            $columnSKS INTEGER
          )
          ''');
  }

  ///TABLE penjualan
  Future<int> insertMahasiswa(PenjualanBarang barang) async {
    var row = {
      columnno: model.no,
      columnnama_barang: model.nama_barang,
      columnjumlah: model.jumlah,
      columnharga: model.harga
    };

    Database db = await instance.database;
    return await db.insert(tablePenjualan, row);
  }

  Future<List<PenjualanBarang>> getPenjualan() async {
    Database db = await instance.database;
    var allData = await db.rawQuery("SELECT * FROM $tablePenjualan");

    List<PenjualanBarang> result = [];
    for (var data in allData) {
      result.add(PenjualanBarang(
          no: data[columnno],
          nama_barang: data[columnnama_barang],
          jumlah: data[columnjumlah],
          harga: int.parse(data[columnharga].toString())));
    }

    return result;
  }

  Future<PenjualanBarang> getPenjualanByno(String nim) async {
    Database db = await instance.database;
    var allData = await db.rawQuery(
        "SELECT * FROM $tablePenjualan WHERE $columnni = $no LIMIT 1");

    if (allData.isNotEmpty) {
      return PenjualanBarang(
          no: allData[0][columnno],
          nama_barang: allData[0][columnnama_barang],
          Jumlah: allData[0][columnjumlah],
          Harga: int.parse(allData[0][columnharga]));
    } else {
      return null;
    }
  }

  Future<int> updateOenjualanBarang(PenjualanBarang Barang) async {
    Database db = await instance.database;
    return await db.rawUpdate(
        'UPDATE $tablePenjualan SET $columnjumlah = ${model.jumlah} '
        'Where $columnno = ${model.no}');
  }

  Future<int> deletePenjualan(String jumlah) async {
    Database db = await instance.database;
    return await db
        .rawDelete('DELETE FROM $tablePenjualan Where $columnno = $);
  }

  clearAllData() async {
    Database db = await instance.database;
    await db.rawQuery("DELETE FROM $tablePenjualan");
  }
}

import 'package:explore_data/useCase/sqflite/PenjualanBarang.dart';
import 'package:explore_data/useCase/sqflite/MySqlflite.dart';
import 'package:flutter/material.dart';

class SqfliteActivity extends StatefulWidget {
  @override
  State<StatefulWidget> createState() => SqfliteActivityState();
}

class SqfliteActivityState extends State<SqfliteActivity> {
  final keyFormPenjualan = GlobalKey<FormState>();

  TextEditingController controllerno = TextEditingController();
  TextEditingController controllernama_barang = TextEditingController();
  TextEditingController controllerjumlah = TextEditingController();
  TextEditingController controllerharga = TextEditingController();

  String no = "";
  String nama_barang = "";
  String jumlah = "";
  int harga = 0;

  List<PenjualanBarang> penjualan = [];

  @override
  void initState() {
    super.initState();

    WidgetsBinding.instance.addPostFrameCallback((_) async {
      Penjualan = await MySqflite.instance.getPenjualan();
      setState(() {});
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
        body: Column(
      crossAxisAlignment: CrossAxisAlignment.stretch,
      children: [
        Container(
          margin: EdgeInsets.only(top: 36, left: 24, bottom: 4),
          child: Text("Input Penjualan",
              style: TextStyle(fontWeight: FontWeight.bold)),
        ),
        Form(
          key: keyFormPenjualan,
          child: Container(
            margin: EdgeInsets.only(left: 24, right: 24),
            child: Column(
              children: [
                TextFormField(
                  controller: controllerno,
                  decoration: InputDecoration(hintText: "no"),
                  validator: (value) => _onValidateText(value),
                  keyboardType: TextInputType.number,
                  onSaved: (value) => no = value,
                ),
                TextFormField(
                  controller: controllerNama_barang,
                  decoration: InputDecoration(hintText: "Nama barang"),
                  validator: (value) => _onValidateText(value),
                  onSaved: (value) => nama_barang = value,
                ),
                TextFormField(
                  controller: controllerjumlah,
                  decoration: InputDecoration(hintText: "jumlah barang"),
                  validator: (value) => _onValidateText(value),
                  onSaved: (value) => jumalah = value,
                ),
                TextFormField(
                  controller: controllerSks,
                  decoration: InputDecoration(hintText: "harga"),
                  validator: (value) => _onValidateText(value),
                  keyboardType: TextInputType.number,
                  onSaved: (value) => harga = int.parse(value),
                ),
              ],
            ),
          ),
        ),
        Container(
          margin: EdgeInsets.only(left: 24, right: 24),
          child: RaisedButton(
            onPressed: () {
              _onSavPenjualan();
            },
            child: Text("Simpan"),
          ),
        ),
        Container(
          margin: EdgeInsets.only(top: 24, left: 24, bottom: 4),
          child: Text("Data Penjualan",
              style: TextStyle(fontWeight: FontWeight.bold)),
        ),
        Expanded(
            child: ListView.builder(
                itemCount: penjualan.length,
                padding: EdgeInsets.fromLTRB(24, 0, 24, 8),
                itemBuilder: (BuildContext context, int index) {
                  var value = Penjualan[index];
                  return Container(
                    margin: EdgeInsets.only(bottom: 12),
                    child: Column(
                      crossAxisAlignment: CrossAxisAlignment.stretch,
                      mainAxisSize: MainAxisSize.min,
                      children: [
                        Text("no: ${value.no}"),
                        Text("nama_barang: ${value.nama barang}"),
                        Text("jumlah: ${value.jumlah}"),
                        Text("harga: ${value.harga}"),
                      ],
                    ),
                  );
                }))
      ],
    ));
  }

  String _onValidateText(String value) {
    if (value.isEmpty) return 'Tidak boleh kosong';
    return null;
  }

  _onSavePenjualan() async {
    FocusScope.of(context).requestFocus(new FocusNode());

    if (keyFormpenjualan.currentState.validate()) {
      keyFormPenjualan.currentState.save();
      controllerno.text = "";
      controllerNama_barang.text = "";
      controllerjumlah.text = "";
      controllerharga.text = "";

      await MySqflite.instance.insertPenjualan(Penjualanbarang(
          no: no, nama_barang: nama_barang, jumlah: jumlah, harga: harga));

      Penjualan = await MySqflite.instance.getPenjualan();
      setState(() {});
    }
  }
}
