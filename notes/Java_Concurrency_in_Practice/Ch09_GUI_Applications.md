#### &#x1F4DA; [Bookshelf](../)
#### &#x1F4DC; [Contents](./README.md#contents)
#### &#x1F448; [Prev](./Ch08_Applying_Thread_Pools.md)
#### &#x1F449; [Next](./Ch10_Avoiding_Liveness_Hazards.md)


## Chapter 09: GUI Applications

- Nearly all GUI toolkits, including Swing and SWT, are implemented as singlethreaded subsystems in which all GUI activity is confined to a single thread.

- In the old days, GUI applications were single-threaded and GUI events were processed from a “main event loop”. Modern GUI frameworks use a model that is only slightly different: they create a dedicated event dispatch thread (EDT) for handling GUI events.

	Multithreaded GUI frameworks tend to be particularly susceptible to deadlock, partially because of the unfortunate interaction between input event processing and any sensible object-oriented modeling of GUI components. Actions initiated by the user tend to “bubble up” from the OS to the application—a mouse click is detected by the OS, is turned into a “mouse click” event by the toolkit, and is eventually delivered to an application listener as a higher level event such as a “button pressed” event. On the other hand, application-initiated actions “bubble down” from the application to the OS—changing the background color of a component originates in the application and is dispatched to a specific component class and eventually into the OS for rendering. Combining this tendency for activities to access the same GUI objects in the opposite order with the requirement of making each object thread-safe yields a recipe for inconsistent lock ordering, which leads to deadlock (see Chapter 10). And this is exactly what nearly every GUI toolkit development effort rediscovered through experience.

- The Swing single-thread rule: Swing components and models should be created, modified, and queried only from the event-dispatching thread.

- As with all rules, there are a few exceptions. A small number of Swing methods may be called safely from any thread; these are clearly identified in the Javadoc as being thread-safe. Other exceptions to the single-thread rule include:
	- SwingUtilities.isEventDispatchThread, which determines whether the current thread is the event thread;
	- SwingUtilities.invokeLater, which schedules a Runnable for execution on the event thread (callable from any thread);
	- SwingUtilities.invokeAndWait, which schedules a Runnable task for execution on the event thread and blocks the current thread until it completes (callable only from a non-GUI thread);
	- methods to enqueue a repaint or revalidation request on the event queue (callable from any thread); and
	- methods for adding and removing listeners (can be called from any thread, but listeners will always be invoked in the event thread).

- From the perspective of the GUI, the Swing table model classes like TableModel and TreeModel are the official repository for data to be displayed. However, these model objects are often themselves “views” of other objects managed by the application. A program that has both a presentation-domain and an applicationdomain data model is said to have a split-model design.

	In a split-model design, the presentation model is confined to the event thread and the other model, the shared model, is thread-safe and may be accessed by both the event thread and application threads. The presentation model registers listeners with the shared model so it can be notified of updates. The presentation model can then be updated from the shared model by embedding a snapshot of the relevant state in the update message or by having the presentation model retrieve the data directly from the shared model when it receives an update event.

	The snapshot approach is simple, but has limitations. It works well when the data model is small, updates are not too frequent, and the structure of the two models is similar. If the data model is large or updates are very frequent, or if one or both sides of the split contain information that is not visible to the other side, it can be more efficient to send incremental updates instead of entire snapshots. This approach has the effect of serializing updates on the shared model and recreating them in the event thread against the presentation model. Another advantage of incremental updates is that finer-grained information about what changed can improve the perceived quality of the display—if only one vehicle moves, we don’t have to repaint the entire display, just the affected regions.

- Thread confinement is not restricted to GUIs: it can be used whenever a facility is implemented as a single-threaded subsystem. Sometimes thread confinement is forced on the developer for reasons that have nothing to do with avoiding synchronization or deadlock. For example, some native libraries require that all access to the library, even loading the library with System.loadLibrary, be made from the same thread.

	Borrowing from the approach taken by GUI frameworks, you can easily create a dedicated thread or single-threaded executor for accessing the native library, and provide a proxy object that intercepts calls to the thread-confined object and submits them as tasks to the dedicated thread. Future and newSingleThreadExecutor work together to make this easy; the proxy method can submit the task and immediately call Future.get to wait for the result.

#### &#x1F4DA; [Bookshelf](../)
#### &#x1F4DC; [Contents](./README.md#contents)
#### &#x1F448; [Prev](./Ch08_Applying_Thread_Pools.md)
#### &#x1F449; [Next](./Ch10_Avoiding_Liveness_Hazards.md)