# Table Component for elisp

`ctable.el` is a table component for emacs lisp. Emacs lisp programs can display a nice table view from an abstract data model.
The many emacs programs have the code for displaying table views, such as `dired`, `list-process`, `buffer-list` and so on. So, ctable.el would provide functions and a table framework for the table views.

# Installation

To use this program, locate this file to load-path directory,
and add the following code to your program code.

```lisp
(require 'ctable)
```

# Quick Start

## Hello World

Giving a list of the rows list to the function `ctbl:popup-table-buffer-easy', a simple table buffer is popped out.

```lisp
(ctbl:popup-table-buffer-easy 
 '((1 2 3 4) (5 6 7 8) (9 10 11 12)))
```

Here is the result image. The header titles are generated automatically.

![sample-1-1](img/sample-1-1.png)


Giving two lists, the latter list is displayed at header titles.

```lisp
(ctbl:popup-table-buffer-easy 
 '((1 2 3 4) (5 6 7 8) (9 10 11 12))
 '(aaa bbb ccc ddd))
```

Here is the result image.

![sample-1-2](img/sample-1-2.png)

## Basic Use

The objects of ctable are designed by the MVC pattern. Programmers can customize ctable objects to use rich table views in the applications easily.

First, one defines the column model and data model for the user application. The former model defines how the column should be display, the latter one does the contents to display.

Second, one chooses builds the view component with the models.

Here is an illustration for the object relations in this basic case.

![Object relations](img/normal_use.png)

Here is a sample code for the model and view.

```lisp
(let* ((column-model ; column model
        (list (make-ctbl:cmodel
              :title "A" :sorter 'ctbl:sort-number-lessp
              :min-width 5 :align 'right)
              (make-ctbl:cmodel
               :title "Title" :align 'center
               :sorter (lambda (a b) (ctbl:sort-number-lessp (length a) (length b))))
              (make-ctbl:cmodel
               :title "Comment" :align 'left)))
       (data
        '((1  "Bon Tanaka" "8 Year Curry." 'a)
          (2  "Bon Tanaka" "Nan-ban Curry." 'b)
          (3  "Bon Tanaka" "Half Curry." 'c)
          (4  "Bon Tanaka" "Katsu Curry." 'd)
          (5  "Bon Tanaka" "Gyu-don." 'e)
          (6  "CoCo Ichi"  "Beaf Curry." 'f)
          (7  "CoCo Ichi"  "Poke Curry." 'g)
          (8  "CoCo Ichi"  "Yasai Curry." 'h)
          (9  "Berkley"    "Hamburger Curry." 'i)
          (10 "Berkley"    "Lunch set." 'j)
          (11 "Berkley"    "Coffee." k)))
       (model ; data model
          (make-ctbl:model
           :column-model column-model :data data))
       (component ; ctable component
        (ctbl:create-table-component-buffer
         :model model)))
  (pop-to-buffer (ctbl:cp-get-buffer component)))
```

Here is the result image.

![sample-2-1](img/sample-2-1.png)


The models have further options and functions to customize the display and behavior, such as column width, text alignment, sorting and so on. (See Model section)

The key-binding on the table can be customized by the keymap object in the usual way. Then, the user program implements the custom function which refers the focused cell. (See Key Bindings section)

The ctable framework provides some hooks to notify the usual events: click, selection change and update view. (See Event Handling section)

The appearance of the table can be customized, such as foreground and background color, tabular lines. (See Display Parameter section)

![ctable components](img/objects.png)


# Advanced Topics

## Column Model

The struct `ctbl:cmodel` is a data type defined by cl-defstruct. This model defines how to display the content along with the each column.

Here is the details of the slot members of `ctbl:cmodel`.

|slot | name description |
|-----|------------------|
|title | **[required]** column header title string. |
|sorter | sorting function which transforms a cell value into sort value. It should return -1, 0 and 1. If nil, `ctbl:sort-string-lessp` is used. |
|align | text alignment: `left`, `right` and `center`. (default: `right`) |
|max-width | maximum width of the column. if `nil`, no constraint. (default: `nil`) |
|min-width | minimum width of the column. if `nil`, no constraint. (default: `nil`) |
|click-hooks | header click hook. a list of functions with two arguments the `ctbl:component` object and the `ctbl:cmodel` one. (default: '(ctbl:cmodel-sort-action)) |

## Data Model

The struct `ctbl:model` is a data type defined by cl-defstruct. This model defines contents to display with column models.

Here is the details of the slot members of `ctbl:model`.

|slot | name description |
|-----|------------------|
|data | **[required]** Table data as a list of rows. A row contains a list of columns. |
|column-model | **[required]** A list of column models. |
|sort-state | The current sort order as a list of column indexes. The index number of the first column is 1. If the index is negative, the sort order is reversed. |

(In the current implementation, the data slot should be statically defined.)

## Key Bindings

The keymap `ctbl:table-mode-map` is used as a default keymap on the table. This keymap is a customization variable for the end users, so it should not be modified by applications.

The component functions `ctbl:create-table-component-buffer` and `ctbl:open-table-buffer` receive a `custom-map` argument to override the keymap on the table buffer. Because the functions connect the given keymap to the default keymap `ctbl:table-mode-map` as parent, application program may define the overriding entries.

The component function `ctbl:create-table-component-region` receives a `keymap` argument to define the keymap on the each characters in the table region.

The ctable framework provides some hooks for the usual event cases. In such cases, the application should use the event handlers, instead of defining the keymap. See the next section.

## Event Handling

The ctable component provides some hooks for the particular events: clicking, selection changing and updating view. The application program can implement some actions without defining keymaps.

Here is a sample code for the click action:

```lisp
(ctbl:cp-add-click-hook 
 cp (lambda () (message "CTable : Click Hook [%S]" 
        (ctbl:cp-get-selected-data-row cp))))
```

where, `cp` is an instance of ctable component. The function `ctbl:cp-add-click-hook` adds the given function as an event handler to the component instance. Here are event handler functions:

- `ctbl:cp-add-click-hook` : on click
- `ctbl:cp-add-selection-change-hook` : on selection change
- `ctbl:cp-add-update-hook` : on update view

The function `ctbl:cp-get-selected-data-row` returns a row object which is defined by the model.
Some component access functions are useful for the action handlers.

- `ctbl:cp-get-selected` : returns a Cell-ID object which is currently selected, such as (1 . 2).
- `ctbl:cp-get-selected-data-row` : returns a row data which is currently selected.
- `ctbl:cp-get-selected-data-cell` : return a cell data which is currently selected.

## Display Parameter

The ctable view renders the tabular form with many rendering parameters. The parameters are set at the slot members of the cl-defstruct `ctbl:param`.

To customize the parameters, one should copy the default parameters like `(copy-ctbl:param ctbl:default-rendering-param)` and set parameters with setter functions. Here is a sample code for parameter customize.

```lisp
  (let ((param (copy-ctbl:param ctbl:default-rendering-param)))
    (setf (ctbl:param-fixed-header param) t)
    (setf (ctbl:param-hline-colors param)
          '((0 . "#00000") (1 . "#909090") (-1 . "#ff0000") (t . "#00ff00")))
    (setf (ctbl:param-draw-hlines param)
          (lambda (model row-index)
            (cond ((memq row-index '(0 1 -1)) t)
                  (t (= 0 (% (1- row-index) 5))))))
    (setf (ctbl:param-bg-colors param)
          (lambda (model row-id col-id str)
            (cond ((string-match "CoCo" str) "LightPink")
                  ((= 0 (% (1- row-index) 2)) "Darkseagreen1")
                  (t nil))))
    ...
    )
```

Here is the details of the slot members of `ctbl:param`.

|slot | name description |
|-----|------------------|
|display-header | if t, display the header row with column models. |
|fixed-header   | if t, display the header row in the header-line area. |
|bg-colors      | '(((row-id . col-id) . colorstr) (t . default-color) ... ) or (lambda (model row-id col-id) colorstr or nil) |
|vline-colors   | "#RRGGBB" or '((0 . colorstr) (t . default-color)) or (lambda (model col-index) colorstr or nil) |
|hline-colors   | "#RRGGBB" or '((0 . colorstr) (t . default-color)) or (lambda (model row-index) colorstr or nil) |
|draw-vlines    | 'all or '(0 1 2 .. -1) or (lambda (model col-index) t or nil ) |
|draw-hlines    | 'all or '(0 1 2 .. -1) or (lambda (model row-index) t or nil ) |
|vertical-line | vertical line character |
|horizontal-line | horizontal line character |
|left-top-corner   | corner character |
|right-top-corner  | corner character |
|left-bottom-corner  | corner character |
|right-bottom-corner  | corner character |
|top-junction | junction character |
|bottom-junction | junction character |
|left-junction | junction character |
|right-junction | junction character |
|cross-junction | junction character |

## View Components

todo...

## Package Installation

todo...

* * * * *

(C) 2012,2013  SAKURAI Masashi  All rights reserved.
m.sakurai at kiwanami.net
