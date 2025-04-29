CompletableFuture Eaxmple with exception handling


import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutionException;

public class CompletableFutureExample {

    public static void main(String[] args) {
        CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
          
            if (true) {
                throw new RuntimeException("Something went wrong!");
            }
            return "Hello, World!";
        })
        .exceptionally(ex -> {
            System.out.println("Exception occurred: " + ex.getMessage());
            return "Default Value";
        });

        try {
            String result = future.get(); 
            System.out.println("Result: " + result);
        } catch (InterruptedException | ExecutionException e) {
            System.out.println("Failed to retrieve result: " + e.getMessage());
        }
    }
}



Flow Explanation:


Async Task Starts:

CompletableFuture.supplyAsync(...) runs the lambda in a separate thread.

Inside the lambda: throw new RuntimeException("Something went wrong!"); is executed.

Exception is Thrown:

The exception interrupts the normal execution of the lambda.

Instead of propagating up and crashing the program, it triggers the next chained method.

Exception Handling with .exceptionally(...):

Since the async task threw an exception, the exceptionally block is invoked.

It prints:

Exception occurred: Something went wrong!
Then it returns a fallback string "Default Value" as the final result of the future.

Blocking Call to .get():

future.get() waits for the computation to finish (which is already handled).

It returns "Default Value" because that’s what was returned from the exceptionally block.

Final Output Printed:

The value is printed:
