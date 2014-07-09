jQueryTaskPromise
=================

This is a queue system to fascilitate ordered operations. It has a configurable retry mechanism.

For detailed examples of use; please see https://github.com/LaskonSoftware/NWGatewayAutomation

Overview:

Create a class to coordinate related operations

```javascript
(function($){
    
    /*
        Define a custom class
    */
    var CustomClass = function(params, go, here){
        this.params = params;
        this.go = go;
        this.here = here;
    };


    /*
        Each atomic operation should be defined in it's own prototype function.
    */

    /*
        There needs to be a method to start ever
    */
    CustomClass.prototype.intialization_step = function(){
        var self = this;
        //Set up the task
        var task = $.task.create(this.step_one.bind(this.step_one));
        
        //Queue up some tasks
        task.then(this.step_two.bind(this));
        task.then(this.step_three.bind(this));

        return task;
    };

    /*
        The first queued up operation
    */
    CustomClass.prototype.step_one = function(task){

        var needs_to_retry;
        //Do something to set needs_to_retry

        if(needs_to_retry){
            return {
                error:true, //This will cause this operation to run again
                delay:3000  //This is how long (in milliseconds) to wait before running again
            };
        }

        return {
            error:false,    //Can also be left out
            delay: 0        //Things worked, next step
        }

    };

    CustomClass.prototype.step_two = function(task){

        var forNextOperation;
        //Something to set forNextOperation

        return {
            error:false,
            delay: 100,//Sometimes a small pause is nice
            args:[forNextOperation]//This will pass this as a param to the next operation
        };
    };

    CustomClass.prototype.step_three = function(fromPrevOperation, task){

        //the passed along args is an array
        if(fromPrevOperation[0].isTrue){//It's an example; deal with the broken
            task.then(this.step_was_true.bind(this));
        }
        else{
            task.then(this.step_final.bind(this));
        }

        return {};//can be empty
    };

    CustomClass.prototype.step_was_true = function(task){
        
        //Here we see how to queue up the same operation to, essentially, start over
        var new_task = this.intialization_step();
        new_task.start_in(delay);//Set some delay before it tries to run again

        task.finish();//Finishes the current task and will proceed with the items in the queue; if there were no others, new_task will start after 'delay'
    };
    
    CustomClass.prototype.step_final = function(task){
        task.finish();//The next item in the queue will be processed
    };


    $.extend(true, $.your_namespace, {
        customClass: {
            create:function(params, go, here){
                return new CustomClass(params, go, here);
            }
        }
    });    

})(jQuery);
```


This will be invoked by something like
```javascript
(function($){
    var myTask = $.your_namespace.customclass.create(stuff, goes, here);
    var task = myTask.initialization_step();
    task.progress();
}(jQuery));
```

