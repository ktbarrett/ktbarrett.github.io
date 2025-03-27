
## stderr is shared in a pipeline

`stderr` in a pipeline is shared between all the programs in the pipeline.
This has some interesting implications.

Imagine you have two programs in a pipeline, both are outputting diagnostic messages to `stderr` at the same time.
Every message program A `write()`s will "immediately" appear to the user.
Before program A has a chance to `write()` another message, program B may `write()` a message of their own.
Thus, diagnostic messages of the two programs will be interleaved and it won't be immediately obvious which program emitted which messages.
So this means we may want to add the program name to diagnostic messages to help disambiguate.

If you `write()` any less than a full message to `stderr` you could even interleave *within* messages.
Big yikes.
On Linux in C environments `stderr` is unbuffered, so every `fputc(c, stderr)` may appear interleaved with another program.
This means we really only want to `write()` a full diagnostic message at a time.

I think this clearly tells us that using typical C standard I/O functions on `stderr` is maybe not advisable,
and instead we really need a message-oriented logging system which includes which program the message originated from.

### Questions...

Other languages, e.g. Python, use line buffering for `stderr` with the assumption that messages will be line-oriented.
That is a good assumption to make, but it doesn't help if you have messages with multiple lines as they may get interleaved and it becomes not immediately obvious which line the following line is associated with.
I have seen some OS startup scripts add line counters within multi-line messages (usually in the format `(2/17)`) to allow the user to manually stitch together multiline diagnostic messages from a single program.

How do libraries log?
Wouldn't they need to know the program name executing them to be able to produce correct log messages?
Some libraries support setting the program name, but it's a little ad-hoc.
Should the ISO C standard include a facility to keep the program name (or perhaps the entire argc/argv) available globally and statically?
This is done in Ada for instance.

One more little thought experiment...
What happens if there are two instances of `cut` running in a pipeline?
How do you differentiate them?
Should POSIX include facilities to name instances of programs?

## Logging levels

Many logging facilities include logging levels.
What is the purpose of the logging levels?
What number of levels are sufficient to reach the intended goal?

My expectation is that including multiple logging levels exist to achieve 2 goals:
1. Label messages consumed by the user or a programmatic filter as to the *severity* of the failure (if it is a failure).
2. Create a hierarchy of verbosity levels (if it is not a failure).

Falling out of this we see the same logging (sometimes known as "severity") levels in Python, VHDL, and many logging libraries for C and C++
(spdlog, easyloggingpp, loguru, plog, to name a few).
They are usually:

* FATAL/PANIC: An unrecoverable program error.
* ERROR: A recoverable or unrecoverable user error.
* WARN: A recoverable user error, or a warning about a feature change or deprecation.
* INFO: Feel good messages showing the user the program is doing something.
* DEBUG/VERBOSE: Messages for debugging a program without the use of a debugger.

The need to label messages according to severity follows on from the classification on types of errors.
Read [this](http://joeduffyblog.com/2016/02/07/the-error-model/) for more info on that.
I think that article does a good enough job I don't need to explain or justify further.

The hierarchy of verbosity levels is very traditional; many Unix tools come with a "verbose" flag (typically `-v`).
Sometimes these tools come with even more verbose levels than that (see the joke about `-vvvvvvvvvvvv` from above).

Is the severity/logging level paradigm really a good one?
I don't think so.
Why would anyone want to set the level above WARN or ERROR so they don't see what they are doing wrong?
If you don't want to see errors and only want to see the programmatic output then redirect `stderr` to a file or `/dev/null` or something.

I don't understand the need to create more than two verbosity levels: normal and debugging.
You have to ask why a user would every need to see verbose information about a program running and in my mind the only answer is for debugging.
In that case, you really want as much information to help with debugging as possible.
The `TRACE` level, ostensibly for tracing the control of the program is a particularly bad idea;
at that point, please use a debugger;
shipping debug symbols isn't as expensive as they used to be.

I think the hierarchy of labels is good, but I don't see the value in changing the log level beyond the default (`INFO`) and debugging (`DEBUG`/`VERBOSE`).
`WARN`, `ERROR`, and certainly `FATAL`/`PANIC` should *always* be output on `stderr`.
Golang seems to agree with me;
their `log` standard library includes only `Panic`, `Fatal`, and `Print`;
no separation of verbosity or even a distinction between normal informational prints (`INFO`) and user-caused errors (`WARN` or `ERROR`).

Finally, I think the logging verbosity level should be able to be set via environment variable, not via the typical verbosity or log level argument (`-v`).
1. This is easier to implement: no argument parsing necessary.
2. This would make setting the verbosity level for an entire pipeline easy.

Perhaps this is something that could be standardized by POSIX...

## Process substitution and log filtering

## Conclusions

So working through the problem step-by-step we came to some reasonable conclusions...
1. `stderr` is for **all** *diagnostic* outputs, while `stdout` is for *programatic* outputs
2. `stderr` is shared between all programs running in a pipeline, so diagnostic messages...
    1. should be written all at once to prevent interleaving messages between multiple programs.
    2. should include a header which includes the program's name.
3. There should be multiple log levels representing severity and cause of error.
    1. The five common log levels: FATAL, ERROR, WARN, INFO, DEBUG are sufficient.
4. All programs --- interactive or not --- should segregate diagnostic and programmatic outputs.
