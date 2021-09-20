# Simple Groovy Logger Without Any Logging Framework


If you need to use a simple logging in your Groovy script you can use some existing and [supported](http://docs.groovy-lang.org/latest/html/api/groovy/util/logging/package-summary.html) frameworks.

Sometimes you do not want or cannot use any other dependencies in your script, so you can write your own simple logger using `methodMissing` implementation:

```groovy
/**
 * Possibility to log TRACE, DEBUG, INFO, WARN and ERROR levels
 * Simply call methods like info, debug etc. (case insensitive)
 * Possibility to set/change:
 * * logFile - location of a log file (default value:default.log)
 * * dateFormat - format of a date in log file(default value:dd.MM.yyyy;HH:mm:ss.SSS)
 * * printToConsole - whether a message should be printed to console as well (default value:false)
 * @author pavel.sklenar
 *
 */
class Logger {
    private File logFile = new File("default.log")
    private String dateFormat = "dd.MM.yyyy;HH:mm:ss.SSS"
    private boolean printToConsole = false
 
    /**
     * Catch all defined logging levels, throw  MissingMethodException otherwise
     * @param name
     * @param args
     * @return
     */
    def methodMissing(String name, args) {
        def messsage = args[0]
        if (printToConsole) {
            println messsage
        }
        SimpleDateFormat formatter = new SimpleDateFormat(dateFormat)
        String date = formatter.format(new Date())
        switch (name.toLowerCase()) {
            case "trace":
                logFile << "${date} TRACE ${messsage}\n"
                break
            case "debug":
                logFile << "${date} DEBUG ${messsage}\n"
                break
            case "info":
                logFile << "${date} INFO  ${messsage}\n"
                break
            case "warn":
                logFile << "${date} WARN  ${messsage}\n"
                break
            case "error":
                logFile << "${date} ERROR ${messsage}\n"
                break
            default:
                throw new MissingMethodException(name, delegate, args)
        }
    }
}
```

Supported log levels are TRACE, DEBUG, INFO, WARN and ERROR (otherwise an exception is thrown). You can simply call these methods (case insensitive) on the instance of the Logger class:

```groovy
def logger = new Logger()
logger.trace "Trace level test"
logger.debug "Debug level test"
logger.info "Info level test"
logger.warn "Warn level test"
logger.error "Error level test"
```

Output:

```bash
23.07.2015;18:42:53.960 TRACE Trace level test
23.07.2015;18:42:54.028 DEBUG Debug level test
23.07.2015;18:42:54.029 INFO  Info level test
23.07.2015;18:42:54.030 WARN  Warn level test
23.07.2015;18:42:54.031 ERROR Error level test
```

You are able to customize your logging with these properties:

- **logFile** - a location of a log file (default value:new File("default.log"))
- **dateFormat** - a date format in a log file (default value:dd.MM.yyyy;HH:mm:ss.SSS)
- **printToConsole** - whether a message should be printed to a console as well, i.e. in addition to a log file (default value:false)

