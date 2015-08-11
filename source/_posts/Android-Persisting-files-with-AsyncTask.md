title: 'Android: Writing and reading files with AsyncTask'
date: 2015-08-12 08:09:53
tags:
- Android
- AsyncTask
- Retrofit
- Gson
---
I would like to share an interesting approach to the use of `AsyncTask` for long running operations that [I bumped into](http://stackoverflow.com/a/6312491/495673) a few days ago. As an example, I will show you how to write to and read from a file while still handling the possible exceptions transparently.

AsyncTaskResult
---------------

First, let's create a generic class that allows us to wrap the results of an `AsyncTask`.

```Java
public class AsyncTaskResult<T> {
    private Exception error;
    private T result;

    public AsyncTaskResult(T result) {
        this.result = result;
    }

    public AsyncTaskResult(Exception error) {
        this.error = error;
    }

    public Exception getError() {
        return error;
    }

    public T getResult() {
        return result;
    }

    public boolean isError() {
        return this.error != null;
    }
}
```

`T` represents the type of the expected result. The method `isError()` will help us know if the AsyncTask finished properly or not. We could also add a second generic type if we expect a particular type of `Exception` to be returned (e.g.: `IOException`).

Writing to a file
-----------------

Now, let's create an AsyncTask which will write some data to a file and return a result.

```Java
new AsyncTask<Response, Void, AsyncTaskResult>() {

    @Override
    protected AsyncTaskResult doInBackground(Response... responses) {
        try {
            BufferedInputStream input = new BufferedInputStream(responses[0].getBody().in());
            FileOutputStream output = mContext.openFileOutput(FILE_PATH, Context.MODE_PRIVATE);
            byte[] data = new byte[1024];
            int count;
            while ((count = input.read(data)) != -1) {
                output.write(data, 0, count);
            }
            output.close();
            input.close();
            return new AsyncTaskResult(null);
        } catch (IOException e) {
            Log.d(TAG, "I/O Exception.", e);
            return new AsyncTaskResult(e);
        }
    }

    @Override
    protected void onPostExecute(AsyncTaskResult result) {
        if (result.isError()) {
            Log.e(TAG, "I/O Exception.", result.getError());
            onError(result.getError()); // Your method to handle the error.
        } else onSuccess(); // Yout method to handle success.
    }
}.execute(response);
```

In the example above, the `AsyncTask` takes a `Response` from a network call (in this case a [Retrofit](http://square.github.io/retrofit/) `Response` object) and returns an `AsyncTaskResult`. `FILE_PATH` is a constant that represents the path of the file. `mContext` is a reference to an Android `Context`, which we will need to perform File I/O operations. Notice I did not assign a type to the `AsyncTaskResult`. In this case we only want to know if the transaction was successful. In that case, we pass `null` to its constructor. `onError` and `onSuccess` are methods you need to implement to handle the possible results.

Reading from a file
-------------------

Finally, it's time to read from the file we created/updated. Let's create another `AsyncTask` which will read from that file and return a result. We will assume our file contains a JSON string that we can parse into a `MyDTO` object.

```Java
new AsyncTask<Void, Void, AsyncTaskResult<MyDTO>() {

    @Override
    protected AsyncTaskResult<MyDTO> doInBackground(Void... voids) {
        try {
            InputStream input = mContext.openFileInput(FILE_PATH);
            InputStreamReader reader = new InputStreamReader(input);
            MyDTO myDTO = mGson.fromJson(reader, MyDTO.class);
            reader.close();
            input.close();
            return new AsyncTaskResult<>(myDTO);
        } catch (IOException e) {
            return new AsyncTaskResult<>(e);
        }
    }

    @Override
    protected void onPostExecute(AsyncTaskResult<MyDTO> result) {
        if (result.isError()) {
            onError(result.getError()); // Your method to handle the error.
        } else onSuccess(result.getResult()); // Your method to process the result.
    }
}.execute();
```

`mGson` is a reference to a [Gson](https://github.com/google/gson) object which we use to parse the content of the JSON file. You could read any other sort of file using a similar approach. In this case, `onSuccess` receives the contents of the file in as a `MyDTO` object.

Conclusion
----------

There are many ways of running time consuming tasks outside the UI thread. `AsyncTask` is a well known utility class provided by the Android framework. By wrapping the result into `AsyncTaskResult` we can elegantly handle success and failure scenarios.

I hope you find this tip useful. Happy coding!
