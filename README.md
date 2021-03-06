[Go FLTK](http://github.com/zot/go-fltk)
========================================

This is a simple go wrapper for [FLTK2](http://www.fltk.org/doc-2.0/html/index.html), which I did to support my Go version of Ober (based on Acme), [Gober](http://github.com/zot/Gober).  It's very small and should be fairly easy to extend to support whatever other widgets and functionality you need.  Please contribute changes you make.  The code is licensed under the very permissive ZLIB license.  FLTK2 is only somewhat supported, but I like its mouse-text behavior better than FLTK 1.3's.


GOALS
=====
* *Good integration with Go* -- the way this binding processes events lends itself well to Go programs (I think).  since there is not call-in from C to Go, only call-out from Go to C, this runs FLTK in a separate thread in order to allow Go to intercept every event.  Go FLTK operations are protected so that they are safe to use in goroutines.
* *Simplicity* -- I'm trying to keep the code small, mostly in Go, generic (multipurpose, but not fat), easy to modify, and easy to extend
* *Small and relatively stand-alone* -- I wanted a toolkit that only depended on basic X functionality and was relatively small so I could statially link it into the Go package.  At this point, fltk.a is 121K and cgo_fltk.so is 312K.  Yes, I'm sitting here with a straight face and claiming that this is "small."  Widget kits are usually several megabytes, so this is small.  Maybe someday this will even be a service instead of a library.


LIMITED FUNCTIONALITY
=====================
This wrapper is far from complete, and I'm only planning on supporting what I need to do Gober, but I welcome additions to it.  In fact, at this point it hardly does anything.  I'll flesh things out as I need them for Gober.


USING Go FLTK
=============
* fltk.Run(while func() bool) runs FLTK until while() returns false.  Run() is defined like this:
	for (while()) {ContinueEvent(Handle(ReadEvent()))}
* fltk.ReadEvent() returns the next *Event structure read
* fltk.Handle(event *Event) runs the widget's event handler (if there is one) and returns what the handler returned (1 means the widget used the event, 0 means it did not)
* fltk.ContinueEvent(status int) allows FLTK to continue processing.  If you do not call this, the GUI will hang.

By default, widgets have empty handlers, so you can treat fltk as an event stream if you like, instead of using event handlers (wait for an event, process the event yourself), which may lend itself well to using Go channels, since Go FLTK operations are safe to use in goroutines.  That boils down to four ways you can deal with events:

	case 1, process an event after the widget handles it:

		user event --> widget --> go event handler

	case 2, process an event before the widget handles it (event stealing):

		user event --> go event handler [optionally: --> widget]

	case 3, roll your own, with default widget behavior (without an event handler):

		user event --> widget --> roll your own with the current event

	case 4, roll your own, with no default widget behavior (stealing without an event handler):

		user event --> roll your own with the current event [optionally: --> widget]

You can install an event handler with widget.SetEventHandler() like this:

    input.SetEventHandler(func(e fltk.Event){println(e.Event)}

Normally, the widget processes the event before Wait returns but you can "steal" events so that your event handler will get the event before the widget has a chance to process it.  If you want to steal events, call widget.StealEvents() with a bit mask for the FLTK event types.  There are constants for the event types and masks for them by appending _MASK to the name, like PUSH_MASK, which is just 1 << PUSH.  You call StealEvents() like this:

   input.StealEvents((1 << PUSH_MASK) | (1 << DRAG_MASK))

There is a bool field in Event, called "Stolen" to tell you whether the event was stolen.  If you want the widget to process the stolen event, just call widget.ContinueEvent().  Here is an example of an event handler that prints a message for stolen push and drag events and then continues them:

	i.StealEvents(fltk.PUSH_MASK | fltk.DRAG_MASK)
	i.SetEventHandler(func(e fltk.Event) {
		if (e.Stolen) {
			println("CONTINUING EVENT: " + strconv.Itoa(e.Event))
			i.ContinueEvent()
		}
	})

There is an example program, test.go, in the "examples" directory, that steals events from a text field, like this.


DEPENDENCIES
============
The make file SHOULD auto-download and unpack FLTK2 for you.  When it builds the Go library, it statically links FLTK into the library so that the only dependencies are -lXext -lXinerama -lXft -lX11 -lXi -lm.

This only runs under X, because of the way I build it, but FLTK purports to run on windows and macs, so if you can test that and help with the Makefile, I'd appreciate it.


BUILDING
========
To build, run "make" in the top-level directory.  This assumes you have g++ installed, because FLTK is written in C++ and you need to use g++ to link the library so it can resolve mangled names.

To install the library, run "make install".

To build the examples, run "make examples".

If you want to eliminate the fltk-library from your directory, you can run "make clean-fltk".


EXAMPLES
========
There are two examples: hello and text.  Hello is the Hello World program from the [FLTK docs](http://www.fltk.org/doc-2.0/html/example1.html) and text is a window with a text field in it that steals the PUSH and DRAG events, printing a message and continuing the events.
