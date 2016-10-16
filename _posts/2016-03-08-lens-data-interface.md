---
title: Creating a Lens for an Object to Provide a New Public Interface
created_at: 2016-03-08 07:18:51 +0100
kind: worklog
tags: [ functional-programming, lens, value ]
vgwort: http://vg01.met.vgwort.de/na/ce56f681d35545b0a450518f8237eb73
comments: on
---

I'm back from my vacation and very deep into programmin already. Before I left, my subconscious mind brought up the notion of a Lens (functional programming) while I was modelling tabular data in Swift and I think this is a very cool approach to keep value types clean and easy to test. I use lenses in my current project to provide a specialized interface to operate with the table. The lenses provide access to column and row iterators, independent of the table's underlying data structure. This way the table doesn't have to worry about all this stuff.

The table is a collection of data. Its data is either organized in rows, columns, or cells as the fundamental unit. Whatever the fundamental unit is, either enumerating rows or columns requires to create a new representation of the data. That's where my not-quite-functional-programming-but-close Lenses come into play. They pull this out of the table.

Let's assume the table is organized by columns as the fundamental unit:

    #!swift
    protocol TableRepresentation {
        var columns: [Index : TableColumn] { get }
    }

    struct Table: TableRepresentation {
        private(set) var columns: [Index : TableColumn]
        
        // ...
    }

`TableColumn` is like an array of cell data with some benefits. `Index` is a positive non-zero integer (1, 2, ..., `UInt.max`). It's what array indexes are supposed to be but never were: human readable and less error-prone. It's also used to print row numbers so I don't have to `"\(index+1)"` in the view.

Enumerating or subscripting the rows, for example for printing purposes, requires a rotation of the data, so to speak: iterate all columns and collect the cells at a specific row index.

Instead of jamming all that into the `Table` itself (which I did at first but abandoned when I found I had to duplicate a lot of stuff during tests), let's create a lens to encapsulate the reading algorithm:

    #!swift
    extension Table {
        var rows: TableRowLens {
            return TableRowLens(table: self)
        }
    }
    
    struct TableRowLens {
        let table: TableRepresentation

        init(table: TableRepresentation) {
            self.table = table
        }
    
        subscript (rowIndex: Index) -> [Index: CellData] {
            return table.columns
                .flatMapDictionary { ($0.0, $0.1.cell(row: rowIndex)) }
        }
    }
    
`flatMapDictionary` is simple: it takes a key-value tuple and when the value is nil, it doesn't try to insert it into the dictionary but skips the entry. It is not part of the Swift stdlib. Here's my humble approach:

    #!swift
    extension Dictionary {

        func flatMapDictionary<K, V>(@noescape transform: (Element) throws -> (K, V?)) rethrows -> [K : V] {

            var result = [K : V]()

            for element in self {
                let (key, value) = try transform(element)

                if hasValue(key) {
                    result[key] = value
                }
            }
        
            return result
        }
    }

The lens is easy to test: just give it a `TableRepresentation` with some column-based data and see whether the result is what you expect:

    #!swift
    func testSubscript_1x1Table_FirstRow_ReturnsRowAt1stIndex() {

       let table = TestTable()
       table.columns = [Index(1)! : column(column: 1, row: 1, text: "content")]

       let lense = TableRowLens(table: table)

       XCTAssertEqual(lense[Index(1)!].count, 1)
       XCTAssertEqual(lense[Index(1)!][Index(1)!], .Text("content"))
    }
   
    class TestTable: TableRepresentation {    
        var columns: [Index : TableColumn] = [:]
    }

    func column(column column: UInt, row: UInt, text: String) -> TableColumn {

        var col = TableColumn()
        col.insert(cell: NewCell(column: Index(column)!, row: Index(row)!, data: .Text(text)))

        return col
    }

See that I used `NewCell` as a value object to hold on to coordinates and cell data? It seems I can never have too many tuples or value types after the switch from Objective-C to Swift, where defining a new type is so cheap.

There are [some Swift μFrameworks for lenses](https://pinboard.in/u:divinedominion/t:lens) I'm aware of, but this simple read-only approach already did its job well. Giving it the ability to change the table would be awesome, too, but I don't know whether I need that kind of flexibility right now. The interface will look nice, but I think the app works really well with `Table.insert(cell: NewCell)` instead of reaching deep inside with `Table.rows[y][x] = cellData`.

For now I need the lense only to check that importing CSV data works. Later feature additions for storing table data may use a different mechanism and make this obsolete. We'll see.

I'm open for corrections of my use of the term. Swift's `String` type has a `CharacterView` that seems to do a similar thing, only I didn't want to use the "view" postfix anywhere except UI. 
