# NewThread_Android
There are some ways to run task in new thread in Android and Java

## AsyncTask 
AsyncTask is one of the easiest ways to implement parallelism in Android without having to deal with more complex methods like Threads. Though it offers a basic level of parallelism with the UI thread, it should not be used for longer operations (of, say, not more than 2 seconds).

AsyncTask has four methods:
- onPreExecute()
- doInBackground()
- onProgressUpdate()
- onPostExecute()

where doInBackground() is the most important as it is where background computations are performed.

Here is a sample code outline with explanations:

```java
public class AsyncTaskTestActivity extends Activity {

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);  

        // This starts the AsyncTask
        // Doesn't need to be in onCreate()
        new MyTask().execute("my string parameter");
    }

    // Here is the AsyncTask class:
    //
    // AsyncTask<Params, Progress, Result>.
    //    Params – the type (Object/primitive) you pass to the AsyncTask from .execute() 
    //    Progress – the type that gets passed to onProgressUpdate()
    //    Result – the type returns from doInBackground()
    // Any of them can be String, Integer, Void, etc. 

    private class MyTask extends AsyncTask<String, Integer, String> {

        // Runs in UI before background thread is called
        @Override
        protected void onPreExecute() {
            super.onPreExecute();

            // Do something like display a progress bar
        }

        // This is run in a background thread
        @Override
        protected String doInBackground(String... params) {
            // get the string from params, which is an array
            String myString = params[0];

            // Do something that takes a long time, for example:
            for (int i = 0; i <= 100; i++) {

                // Do things

                // Call this to update your progress
                publishProgress(i);
            }

            return "this string is passed to onPostExecute";
        }

        // This is called from background thread but runs in UI
        @Override
        protected void onProgressUpdate(Integer... values) {
            super.onProgressUpdate(values);

            // Do things like update the progress bar
        }

        // This runs in UI when background thread finishes
        @Override
        protected void onPostExecute(String result) {
            super.onPostExecute(result);

            // Do things like hide the progress bar or change a TextView
        }
    }
}
```

## ExecutorService
An Executor that provides methods to manage termination and methods that can produce a Future for tracking progress of one or more asynchronous tasks.
An ExecutorService can be shut down, which will cause it to reject new tasks. Two different methods are provided for shutting down an ExecutorService. The shutdown() method will allow previously submitted tasks to execute before terminating, while the shutdownNow() method prevents waiting tasks from starting and attempts to stop currently executing tasks. Upon termination, an executor has no tasks actively executing, no tasks awaiting execution, and no new tasks can be submitted. An unused ExecutorService should be shut down to allow reclamation of its resources.
Method submit extends base method execute(Runnable) by creating and returning a Future that can be used to cancel execution and/or wait for completion. Methods invokeAny and invokeAll perform the most commonly useful forms of bulk execution, executing a collection of tasks and then waiting for at least one, or all, to complete. (Class ExecutorCompletionService can be used to write customized variants of these methods.)
The Executors class provides factory methods for the executor services provided in this package.
Usage Examples
Here is a sketch of a network service in which threads in a thread pool service incoming requests. It uses the preconfigured newFixedThreadPool(int) factory method:

```java
 class NetworkService implements Runnable {
   private final ServerSocket serverSocket;
   private final ExecutorService pool;

   public NetworkService(int port, int poolSize)
       throws IOException {
     serverSocket = new ServerSocket(port);
     pool = Executors.newFixedThreadPool(poolSize);
   }

   public void run() { // run the service
     try {
       for (;;) {
         pool.execute(new Handler(serverSocket.accept()));
       }
     } catch (IOException ex) {
       pool.shutdown();
     }
   }
 }

 class Handler implements Runnable {
   private final Socket socket;
   Handler(Socket socket) { this.socket = socket; }
   public void run() {
     // read and service request on socket
   }
 }
```

The following method shuts down an ExecutorService in two phases, first by calling shutdown to reject incoming tasks, and then calling shutdownNow, if necessary, to cancel any lingering tasks:

```java
 void shutdownAndAwaitTermination(ExecutorService pool) {
   pool.shutdown(); // Disable new tasks from being submitted
   try {
     // Wait a while for existing tasks to terminate
     if (!pool.awaitTermination(60, TimeUnit.SECONDS)) {
       pool.shutdownNow(); // Cancel currently executing tasks
       // Wait a while for tasks to respond to being cancelled
       if (!pool.awaitTermination(60, TimeUnit.SECONDS))
           System.err.println("Pool did not terminate");
     }
   } catch (InterruptedException ie) {
     // (Re-)Cancel if current thread also interrupted
     pool.shutdownNow();
     // Preserve interrupt status
     Thread.currentThread().interrupt();
   }
 }
```

Memory consistency effects: Actions in a thread prior to the submission of a Runnable or Callable task to an ExecutorService happen-before any actions taken by that task, which in turn happen-before the result is retrieved via Future.get().

or another example of ExecutorService:
```java
ExecutorService executorService = Executors.newFixedThreadPool(1);
        executorService.submit(() -> {
          //Show waiting on UI thread:
          runOnUiThread(() -> progressing(true));
          
          //Run task:
          ...
          
          //Return UI thread to ideal mode:
          runOnUiThread(() -> progressing(false));
        }
```

## Thread
A Thread is a concurrent unit of execution. It has its own call stack for methods being invoked, their arguments and local variables. Each virtual machine instance has at least one main Thread running when it is started; typically, there are several others for housekeeping. The application might decide to launch additional Threads for specific purposes.
Threads in the same VM interact and synchronize by the use of shared objects and monitors associated with these objects. Synchronized methods and part of the API in Object also allow Threads to cooperate.
There are basically two main ways of having a Thread execute application code. One is providing a new class that extends Thread and overriding its run() method. The other is providing a new Thread instance with a Runnable object during its creation. In both cases, the start() method must be called to actually execute the new Thread.
Each Thread has an integer priority that basically determines the amount of CPU time the Thread gets. It can be set using the setPriority(int) method. A Thread can also be made a daemon, which makes it run in the background. The latter also affects VM termination behavior: the VM does not terminate automatically as long as there are non-daemon threads running.

```java
class PrimeRun implements Runnable {
         long minPrime;
         PrimeRun(long minPrime) {
             this.minPrime = minPrime;
         }

         public void run() {
             // compute primes larger than minPrime
              . . .
         }
     }
     
PrimeRun p = new PrimeRun(143);
new Thread(p).start();
```
See more about Thread in android: https://developer.android.com/reference/java/lang/Thread

## Thread with Runnable
Represents a command that can be executed. Often used to run code in a different Thread.

```java
public void onClick(View v) {
        new Thread(new Runnable() {
            public void run() {
                // a potentially time consuming task
                final Bitmap bitmap =
                        processBitMap("image.png");
                imageView.post(new Runnable() {
                    public void run() {
                        imageView.setImageBitmap(bitmap);
                    }
                });
            }
        }).start();
    }
```
or simple:
```java
public void onClick(View v) {
        new Thread(() -> {
            // a potentially time consuming task
            final Bitmap bitmap =
                    processBitMap("image.png");
            imageView.post((Runnable) () -> imageView.setImageBitmap(bitmap));
        }).start();
    }
```

for more datail: https://developer.android.com/codelabs/android-training-create-asynctask
