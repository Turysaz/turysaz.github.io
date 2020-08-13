The following is a textual summary of the gists of @bitbonk.
* https://gist.github.com/bitbonk/b47243e08ef9c262d10e2670a301fa68
* https://gist.github.com/bitbonk/bedfddafb2810c15d59271bdddd98e1a
* https://gist.github.com/bitbonk/2d2b9c94ee475f3d3c725b1006c3e113

Intuitively, if aiming to achieve an sequentially processed chain of async operations,
One might try to do something like the following:

    private void EnqueueNextOperation(Func<Task> next){
        this.chain = this.chain.ContinueWith(previous => await next());
    }

However, that does no make the task getting processed in order.
The problem is, that `ContinueWith` does **not** await the next operation.
Instead, it behaves like this:

On the first call of the method above, the task `this.chain` is indeed run to completion.
After that however, it is replaced by a task that is not awaited but executed as is and
returned immediately.
The result of that task is the inner delegate, that is now running "in the background".

The chain member is set to the outter task, which has returned.
On the next call of `EnqueueNextOperation`, the delegate can again be executed immediately,
because the `this.chain.IsCompleted` is true independently from the actual operation.
To solve this problem, one would have to await the outter task and set `this.chain` to
the inner task, like that:

    this.chain = await this.chain.ContinueWith(...);

That however would require the whole method to be async.
A better solution is to use the `Task.Unwrap` method, that creates a single `Task` of a
`Task<Task>` construct.

Note however, that this does, contraintuitively, NOT extract the inner task. Instead, it
creates a completely new wrapper for the whole thing, that will return when the whole
operation returns.
