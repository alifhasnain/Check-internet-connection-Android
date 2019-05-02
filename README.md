# Check-internet-connection-Android

Bellow Class will handle internet connection availability in background. Create the instance of this class in your activity class as a global variable. 

### CheckInternetConnection.java
```java
import android.os.Handler;
import android.os.Looper;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.net.Socket;
import java.net.SocketAddress;

public class CheckInternetConnection  {

    private ConnectionChangeListener connectionChangeListener;

    private Thread mConnectionCheckerThread;

    private boolean stopThread = false;

    private int mUpdateInterval = 3000;

    public CheckInternetConnection() {

        mConnectionCheckerThread = new Thread(new Runnable() {

            @Override
            public void run() {

                sleep(1000);

                while (!stopThread)    {
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
        });

        mConnectionCheckerThread.setPriority(3);

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
        mConnectionCheckerThread.start();
        stopThread = false;
    }

    public void removeConnectionChangeListener()    {
        stopThread = true;
    }
}

interface ConnectionChangeListener   {
    void onConnectionChanged(boolean isConnectionAvailable);
}
```

Then add `addConnectionChangeListener` to `onStart()` to listen connection change.

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
                    Toast.makeText(MainActivity.this, "No internet connection not available!", Toast.LENGTH_SHORT).show();
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
 
 You should also remove the listener in your `onPause()` and `onStop()` method or it will be keep running on a background thread and waste battery.
 
 ```java
 @Override
    protected void onPause() {
        super.onPause();
        connectionChecker.removeConnectionChangeListener();
    }
 ```
 
