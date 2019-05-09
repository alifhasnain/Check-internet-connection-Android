# Check-internet-connection-Android

Bellow Class will handle internet connection availability in background. Create the instance of this class in your activity class as a global variable. 

### CheckInternetConnection.java
```java
import android.os.Handler;
import android.os.HandlerThread;
import android.os.Looper;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.net.Socket;
import java.net.SocketAddress;

public class CheckInternetConnection  {

    private ConnectionChangeListener connectionChangeListener;

    private HandlerThread mHandlerThread;

    private Handler mConnectionCheckerHandler;

    private int mUpdateInterval = 3000;

    public CheckInternetConnection() {
        initHandler();
        mConnectionCheckerHandler.post(new ConnectionCheckRunnable());
    }

    private void initHandler() {
        mHandlerThread = new HandlerThread("MyHandlerThread");
        mHandlerThread.setPriority(3);
        mHandlerThread.start();
        Looper looper = mHandlerThread.getLooper();
        mConnectionCheckerHandler = new Handler(looper);
    }

    public int getUpdateInterval() {
        return mUpdateInterval;
    }

    public void setUpdateInterval(int updateIntervalInMillis) {
        this.mUpdateInterval = updateIntervalInMillis;
    }

    private void updateListenerInMainThread(final boolean connectionAvailability)   {
        new Handler(Looper.getMainLooper()).post(new Runnable() {
            @Override
            public void run() {
                connectionChangeListener.onConnectionChanged(connectionAvailability);
            }
        });
    }

    private void sleep(int timeInMilli) {

        try {
            Thread.sleep(timeInMilli);
        }
        catch (Exception e1) {
            e1.printStackTrace();
        }
    }

    public void addConnectionChangeListener(ConnectionChangeListener connectionChangeListener) {
        this.connectionChangeListener = connectionChangeListener;
        initHandler();
        mConnectionCheckerHandler.post(new ConnectionCheckRunnable());
    }

    public void removeConnectionChangeListener()    {
        mHandlerThread.quit();
    }

    class ConnectionCheckRunnable implements Runnable  {

        @Override
        public void run() {

            sleep(1000);

            while (true)    {
                try {
                    int timeoutMs = 1500;
                    Socket sock = new Socket();
                    SocketAddress sockaddr = new InetSocketAddress("8.8.8.8", 53);

                    sock.connect(sockaddr, timeoutMs);
                    sock.close();

                    updateListenerInMainThread(true);

                    sleep(mUpdateInterval);

                } catch (IOException e) {

                    updateListenerInMainThread(false);

                    sleep(mUpdateInterval);
                }
            }
        }
    }
}

interface ConnectionChangeListener   {
    void onConnectionChanged(boolean isConnectionAvailable);
}
```

Then add `addConnectionChangeListener` to `onStart()` to listen connection change. By default it will listen to connection change after 3000 millisecond (3 second). But this update interval can be set manually by calling `setUpdateInterval()` method.

```java
.......
.....

private boolean connectionAvailable = true;

........
.....

@Override
    protected void onStart() {
        super.onStart();

        connectionChecker.addConnectionChangeListener(new ConnectionChangeListener() {
            @Override
            public void onConnectionChanged(boolean isConnectionAvailable) {
                if(connectionAvailable && !isConnectionAvailable) {
                    Toast.makeText(MainActivity.this, "Internet connection unavailable!", Toast.LENGTH_SHORT).show();
                    connectionAvailable = false;
                }
                else if(!connectionAvailable && isConnectionAvailable) {
                    Toast.makeText(MainActivity.this, "Internet connection is back again.", Toast.LENGTH_SHORT).show();
                    connectionAvailable = true;
                }
            }
        });
    }
 ```
 
 You should also remove the listener in your `onStop()` method or it will be keep running on a background thread and cause thread leak.
 
 ```java
 @Override
    protected void onStop() {
        super.onStop();
        connectionChecker.removeConnectionChangeListener();
    }
 ```
 
