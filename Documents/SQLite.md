
# SQLite Database
Database.swift

```bash
import Foundation
import SQLite3
import Foundation

class Person
{
    
    var name: String = ""
    var age: Int = 0
    var id: Int = 0
    
    init(id:Int, name:String, age:Int)
    {
        self.id = id
        self.name = name
        self.age = age
    }
    
}
class DBHelper
{
    init()
    {
        db = openDatabase()
        createTable()
    }

    let dbPath: String = "myDb.sqlite"
    var db:OpaquePointer?

    func openDatabase() -> OpaquePointer?
    {
        let fileURL = try! FileManager.default.url(for: .documentDirectory, in: .userDomainMask, appropriateFor: nil, create: false)
            .appendingPathComponent(dbPath)
        var db: OpaquePointer? = nil
        if sqlite3_open(fileURL.path, &db) != SQLITE_OK
        {
            print("error opening database")
            return nil
        }
        else
        {
            print("Successfully opened connection to database at \(dbPath)")
            return db
        }
    }
    
    func createTable() {
        let createTableString = "CREATE TABLE IF NOT EXISTS person(Id INTEGER PRIMARY KEY,name TEXT,age INTEGER);"
        var createTableStatement: OpaquePointer? = nil
        if sqlite3_prepare_v2(db, createTableString, -1, &createTableStatement, nil) == SQLITE_OK
        {
            if sqlite3_step(createTableStatement) == SQLITE_DONE
            {
                print("person table created.")
            } else {
                print("person table could not be created.")
            }
        } else {
            print("CREATE TABLE statement could not be prepared.")
        }
        sqlite3_finalize(createTableStatement)
    }
    
    
    func insert(id:Int, name:String, age:Int)
    {
        let persons = read()
        for p in persons
        {
            if p.id == id
            {
                return
            }
        }
        let insertStatementString = "INSERT INTO person (Id, name, age) VALUES (?, ?, ?);"
        var insertStatement: OpaquePointer? = nil
        if sqlite3_prepare_v2(db, insertStatementString, -1, &insertStatement, nil) == SQLITE_OK {
            sqlite3_bind_int(insertStatement, 1, Int32(id))
            sqlite3_bind_text(insertStatement, 2, (name as NSString).utf8String, -1, nil)
            sqlite3_bind_int(insertStatement, 3, Int32(age))
            
            if sqlite3_step(insertStatement) == SQLITE_DONE {
                print("Successfully inserted row.")
            } else {
                print("Could not insert row.")
            }
        } else {
            print("INSERT statement could not be prepared.")
        }
        sqlite3_finalize(insertStatement)
    }
    
     func update(id:Int, name:String, age:Int){
        let updateStatementString = "UPDATE person SET name = '\(name)', age = '\(age)' WHERE id = '\(id)';"
         var updateStatement: OpaquePointer? = nil
         if sqlite3_prepare_v2(db, updateStatementString, -1, &updateStatement, nil) == SQLITE_OK {
                if sqlite3_step(updateStatement) == SQLITE_DONE {
                  
                       print("Successfully updated row.")
                } else {
                       print("Could not update row.")
                }
              } else {
                    print("UPDATE statement could not be prepared")
              }
              sqlite3_finalize(updateStatement)
    }

    func read() -> [Person] {
        let queryStatementString = "SELECT * FROM person;"
        var queryStatement: OpaquePointer? = nil
        var psns : [Person] = []
        if sqlite3_prepare_v2(db, queryStatementString, -1, &queryStatement, nil) == SQLITE_OK {
            while sqlite3_step(queryStatement) == SQLITE_ROW {
                let id = sqlite3_column_int(queryStatement, 0)
                let name = String(describing: String(cString: sqlite3_column_text(queryStatement, 1)))
                let year = sqlite3_column_int(queryStatement, 2)
                psns.append(Person(id: Int(id), name: name, age: Int(year)))
                print("Query Result:")
                print("\(id) | \(name) | \(year)")
            }
        } else {
            print("SELECT statement could not be prepared")
        }
        sqlite3_finalize(queryStatement)
        return psns
    }
    
    func deleteByID(id:Int) {
        let deleteStatementStirng = "DELETE FROM person WHERE Id = ?;"
        var deleteStatement: OpaquePointer? = nil
        if sqlite3_prepare_v2(db, deleteStatementStirng, -1, &deleteStatement, nil) == SQLITE_OK {
            sqlite3_bind_int(deleteStatement, 1, Int32(id))
            if sqlite3_step(deleteStatement) == SQLITE_DONE {
                print("Successfully deleted row.")
            } else {
                print("Could not delete row.")
            }
        } else {
            print("DELETE statement could not be prepared")
        }
        sqlite3_finalize(deleteStatement)
    }
    
}

```
Viewcontroller.swift
```bash
import UIKit

class ViewController: UIViewController {
    
    var db:DBHelper = DBHelper()
      
      var persons:[Person] = []
    override func viewDidLoad() {
        super.viewDidLoad()
        
//        personTable.register(UITableViewCell.self, forCellReuseIdentifier: cellReuseIdentifier)
//               personTable.delegate = self
//               personTable.dataSource = self
               
               db.insert(id: 1, name: "Bilal", age: 24)
               db.insert(id: 2, name: "Bosh", age: 25)
               db.insert(id: 3, name: "Thor", age: 23)
               db.insert(id: 4, name: "Edward", age: 44)
               db.insert(id: 5, name: "Ronaldo", age: 34)
               db.insert(id: 6, name: "Urvashi", age: 22)
               
               persons = db.read()
        
               db.update(id: 6, name: "urvi", age: 23)
               persons = db.read()
        
        print(persons)
        // Do any additional setup after loading the view.
    }
}
```

