# Scraps and Scripts
Description: Series of stuff I have collected over time that I have felt helpful.

## function_check.php

If you're doing shared hosting, it is extremely likely your provider has some sort of restrictions on the platform for what you can run. You can use this script to test if the function you are trying to use is restricted by that provider:

To dowload it via CLI:

```
curl -o functioncheck.php https://raw.githubusercontent.com/philipjewell/scraps-and-scripts/master/function_check.php
```

Just swap out the `FUNCTION_HERE` in the code with the function you are trying to test and run the script. Since this is a basic if else statement, it will allow you to see if the function is able to fun on the server. 
