---
layout: post
permalink: /angularjs-smart-table-select-all-rows-directive
title: AngularJS Smart-Table select all rows directive
path: 2015-08-02-angularjs-smart-table-select-all-rows-directive.md
---

So I’ve been recently working on an app that required the ability for users to select all rows in a table, as well as selecting rows manually. I then realised that such a function had not existed yet for Smart Table Module, so I thought I’d share this with you guys.
You’ll also be able the get the data from the selected rows into a `$scope`, which for some reason, is not yet implemented in Smart-Table.

> [Smart Table](//google.com) is a table module for AngularJS. It allows you to quickly compose your table in a declarative way including sorting, filtering, row selection and pagination. It’s also lightweight (around 3kb minified) and has no other dependencies than Angular itself.

Firstly, let’s start with the directives. There is one for row slection, which creates the checkbox for every row and assigns the `st-selected` class and another to select all the rows, to which we pass our table data and it checks it to see which items have `isSelected == true`

###### rowSelectAll.js
{% highlight javascript %}
function rowSelectAll() {

  return {
    require: '^stTable',
    template: '<input type="checkbox">',
    scope: {
      all: '=rowSelectAll',
      selected: '='
    },
    link: function (scope, element, attr) {

      scope.isAllSelected = false;

      element.bind('click', function (evt) {

        scope.$apply(function () {

          scope.all.forEach(function (val) {

            val.isSelected = scope.isAllSelected;

          });

        });

      });

      scope.$watchCollection('selected', function(newVal) {

        var s = newVal.length;
        var a = scope.all.length;

        if ((s == a) && s > 0 && a > 0) {

          element.find('input').attr('checked', true);
          scope.isAllSelected = false;

        } else {

          element.find('input').attr('checked', false);
          scope.isAllSelected = true;

        }

      });
    }
  };
}

angular
  .module('app')
  .directive('rowSelectAll', rowSelectAll)
{% endhighlight %}

###### rowSelect.js
{% highlight javascript %}
function rowSelect() {
  return {
    require: '^stTable',
    template: '<input type="checkbox">',
    scope: {
        row: '=rowSelect'
    },
    link: function (scope, element, attr, ctrl) {

      element.bind('click', function (evt) {

        scope.$apply(function () {

            ctrl.select(scope.row, 'multiple');

        });

      });

      scope.$watch('row.isSelected', function (newValue) {

        if (newValue === true) {

            element.parent().addClass('st-selected');
            element.find('input').attr('checked', true);

        } else {

            element.parent().removeClass('st-selected');
            element.find('input').attr('checked', false);

        }
      });
    }
  };
}

angular
  .module('app')
  .directive('rowSelect', rowSelect)
{% endhighlight %}

Next, in order to get data from the selected rows, I’ve created two functions. `selectAll()` goes with the `rowSelectAll` directive and we pass it the entire dataset. `select()` only needs the `row` from `ng-repeat`, being assigned to the single row selection directive.

###### app.js
{% highlight javascript %}
function MainCtrl() { 
  
  // Declare the array for the selected items
  this.selected = []; 
  
  // Function to get data for all selected items
  this.selectAll = function (collection) {
    
    // if there are no items in the 'selected' array, 
    // push all elements to 'selected'
    if (this.selected.length === 0) {
      
      angular.forEach(collection, function(val) {
        
        this.selected.push(val.id); 
        
      }.bind(this));
      
    // if there are items in the 'selected' array, 
    // add only those that ar not
    } else if (this.selected.length > 0 && this.selected.length != this.data.length) {
      
      angular.forEach(collection, function(val) {
        
        var found = this.selected.indexOf(val.id);
        
        if(found == -1) this.selected.push(val.id);
        
      }.bind(this));
      
    // Otherwise, remove all items
    } else  {
      
      this.selected = [];
      
    }
    
  };
  
  // Function to get data by selecting a single row
  this.select = function(id) {
    
    var found = this.selected.indexOf(id);
    
    if(found == -1) this.selected.push(id);
    
    else this.selected.splice(found, 1);
    
  }
}

angular
  .module('app', ['smart-table'])
  .controller('MainCtrl', MainCtrl)
{% endhighlight %}

The directives add the `st-selected` class to every row that is selected, so in order to see the selected row, just style that class.

###### style.css
{% highlight css %}
.st-selected > td {
  background: aliceblue;
}
{% endhighlight %}

### See it in action

<iframe style="width: 100%; height: 500px" src="http://embed.plnkr.co/iIVlfFZpmXrRvc7rLkFf" frameborder="0" allowfullscren="allowfullscren"></iframe>

And that’s it! If you think the above can be improved, let me know in the comments below. Hope this was helpful.