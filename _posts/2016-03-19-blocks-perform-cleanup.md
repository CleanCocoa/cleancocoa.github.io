---
title: "Blocks: Useful to Perform Actions In Context"
created_at: 2016-03-19 14:25:31 +0100
kind: worklog
tags: [ closure, clean ]
comments: on
---


Something short and sweet I want to share with you because I love how it reads:

    #!swift
    func refreshCell(rowIndexes rowIndexes: NSIndexSet, columnIndexes: NSIndexSet) {
        
        preservingSelection {
            tableView.reloadDataForRowIndexes(rowIndexes, columnIndexes: columnIndexes)
        }
    }

`restoringSelection` is a function that takes a block and performs whatever the block does while restoring the current selection in a table view (here: `NSTableView`).

Here's the implementation.

    #!swift
    private func preservingSelection(@noescape block: () -> Void) {

        let oldSelection = selection()
        block()
        selectCells(oldSelection)
    }

    typealias SelectedCells = (columns: NSIndexSet, rows: NSIndexSet)

    private func selection() -> SelectedCells {

        let selectedRows = tableView.selectedRowIndexes
        let selectedColumns = tableView.selectedColumnIndexes

        return (selectedColumns, selectedRows)
    }

    private func selectCells(selection: SelectedCells) {

        tableView.selectRowIndexes(selection.rows, byExtendingSelection: false)
        tableView.selectColumnIndexes(selection.columns, byExtendingSelection: false)
    }

The resulting code from above reads so much nicer than querying `selection()` first and restoring it afterwards. It communicates very clearly that some state from before will be preserved (or restored). 

The little joys.
