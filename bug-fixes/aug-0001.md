# Guess  the Bug 1 

This code looks fine. It isn't.

```
@Composable
fun TaskList(tasks: List<Task>) {
  LazyColumn {
        itemsIndexed(tasks) { index, task -> 
           var isChecked by remember { mutableStateOf(task.isDone) }
           TaskRow( 
              task = task,
              checked = isChecked,
              onCheckedChange = { isChecked = it }
              }
          }
      }
}
```
## What breaks, and When? 


# Hear's what's happening:

`remember` here is keyed only by position in the list, not by the task itself. Delete or reorder and item, and comppose reuses that slot's remenbered state for whatever task now sits that index - checkboxes  end up marked "done" on the wrong task.

Fix: give itemsIndexed (or items) an explicit key = { _, task --> task.id }, so state follows the item, not its position. 

## Before Code:

```
@Composable
fun TaskList(tasks: List<Task>) {
  LazyColumn {
        itemsIndexed(tasks) { index, task -> 
           var isChecked by remember { mutableStateOf(task.isDone) }
           TaskRow( 
              task = task,
              checked = isChecked,
              onCheckedChange = { isChecked = it }
              }
          }
      }
}
```

## After Code 

```
@Composable
fun TaskList(tasks: List<Task>) {
  LazyColumn {
        itemsIndexed(tasks) { index, task -> task.id -> 
           var isChecked by remember { mutableStateOf(task.isDone) }
           TaskRow( 
              task = task,
              checked = isChecked,
              onCheckedChange = { isChecked = it }
              }
          }
      }
}
```
