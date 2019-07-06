---
title: 'js实现sleep'
date: 2019-04-01 15:23:24
tags:
---

这个是在某狗面试里首次遇到的问题（当时只是给出了一个最为简单的答案——伪死循环QAQ）。

由于JS是单线程的，不像其它面向对象的编程语言（java）一样有其内置的 `sleep` 事件。而我们对sleep最直观的的体验应该就是jQuery的 `delay()` 方法了：
``````html
<div id="foo"></div>
<script>
	$('#foo').slideUp(300).delay(800).fadeIn(400);
</script>
``````
``````js
// Based off of the plugin by Clint Helfers, with permission.
// http://blindsignals.com/index.php/2009/07/jquery-delay/
jQuery.fn.delay = function( time, type ) {
	time = jQuery.fx ? jQuery.fx.speeds[ time ] || time : time;
	type = type || "fx";

	return this.queue( type, function( next, hooks ) {
		var timeout = setTimeout( next, time );
		hooks.stop = function() {
			clearTimeout( timeout );
		};
	});
};

jQuery.fn.extend({
	queue: function( type, data ) {
		var setter = 2;

		if ( typeof type !== "string" ) {
			data = type;
			type = "fx";
			setter--;
		}

		if ( arguments.length < setter ) {
			return jQuery.queue( this[0], type );
		}

		return data === undefined ?
			this :
			this.each(function() {
				var queue = jQuery.queue( this, type, data );

				// ensure a hooks for this queue
				jQuery._queueHooks( this, type );

				if ( type === "fx" && queue[0] !== "inprogress" ) {
					jQuery.dequeue( this, type );
				}
			});
	},
	dequeue: function( type ) {
		return this.each(function() {
			jQuery.dequeue( this, type );
		});
	},
	clearQueue: function( type ) {
		return this.queue( type || "fx", [] );
	},
	// Get a promise resolved when queues of a certain type
	// are emptied (fx is the type by default)
	promise: function( type, obj ) {
		var tmp,
			count = 1,
			defer = jQuery.Deferred(),
			elements = this,
			i = this.length,
			resolve = function() {
				if ( !( --count ) ) {
					defer.resolveWith( elements, [ elements ] );
				}
			};

		if ( typeof type !== "string" ) {
			obj = type;
			type = undefined;
		}
		type = type || "fx";

		while ( i-- ) {
			tmp = jQuery._data( elements[ i ], type + "queueHooks" );
			if ( tmp && tmp.empty ) {
				count++;
				tmp.empty.add( resolve );
			}
		}
		resolve();
		return defer.promise( obj );
	}
});
``````

### 手动实现

``````js
function Parent(name) {
	this.name = name;
	this.funList = [];
	this.index = 0;
	this.timer = null;
	this.runIndex = 0;
}

Parent.prototype.sleep = function (time) {
    // 睡眠时间
    if (this.funList[this.index] && typeof this.funList[this.index].fun != 'function') {
        time += this.funList[this.index].time;
    }
    this.funList[this.index] = {
        fun: null,
        time: time
    };
    return this;
}

Parent.prototype.run = function (fun) {
    if (this.funList[this.index] && typeof this.funList[this.index].fun != 'function') {
        this.funList[this.index].fun = fun;
        this.index++;
    } else {
        this.index++;
        this.funList[this.index] = {
            time: 0,
            fun: fun
        };
    }
    console.log(this.funList);
    this.start();
}

Parent.prototype.start = function () {
    var self = this;
    if (this.timer) return;
    var startFn = function (funList, index) {
        if (!funList[index]) {
            clearTimeout(self.timer)
            return false;
        }
        var time = funList[index].time;
        var fun = funList[index].fun;
        typeof fun == 'function' || (fun = function () {});
        self.timer = setTimeout(function () {
            fun();
            self.runIndex++;
            startFn(funList, ++index);
        }, time);
    };
    startFn(this.funList, this.runIndex);
}
       
Parent.prototype.sayHi = function (msg) {
    var fun = function () {
       	if (!msg) return;
       	console.log(msg);
    }
    this.run(fun);
    return this;
}

var start = function (name) {
    console.log(name);
    return new Parent(name);
}

console.log(start('许国前').sleep(2000).sayHi('你好').sleep(2000).sayHi('我不好'));

``````