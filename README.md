# Check-internet-connection-Android

Bellow Class will handle internet connection availablit in background. Just create the instance of this class as global variable. Then add

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

                        sleep(3000);

                    } catch (IOException e) {

                        updateListenerInMainThread(false);

                        sleep(3000);
                    }
                }
            }
        });

        mConnectionCheckerThread.setPriority(3);

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
