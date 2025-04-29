1>CompletableFuture Eaxmple with exception handling


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







2. Write service layer code using completable future.


@Service
public class UserService {

    private final UserRepository userRepository;
    private final OrderRepository orderRepository;

    public UserService(UserRepository userRepository, OrderRepository orderRepository) {
        this.userRepository = userRepository;
        this.orderRepository = orderRepository;
    }

    public CompletableFuture<UserDetailsDTO> getUserDetailsAsync(Long userId) {
        // Fetch user data asynchronously
        CompletableFuture<User> userFuture = CompletableFuture.supplyAsync(() -> userRepository.findById(userId).orElseThrow());

        // Fetch user orders asynchronously
        CompletableFuture<List<Order>> ordersFuture = CompletableFuture.supplyAsync(() -> orderRepository.findByUserId(userId));

        // Combine both results once both are available
        return userFuture.thenCombine(ordersFuture, (user, orders) -> {
            UserDetailsDTO dto = new UserDetailsDTO();
            dto.setUserName(user.getName());
            dto.setEmail(user.getEmail());
            dto.setOrders(orders);
            return dto;
        }).exceptionally(ex -> {
            // Handle any exception and return fallback value or throw a custom exception
            System.out.println("Exception occurred: " + ex.getMessage());
            return new UserDetailsDTO(); // return empty or default object
        });
    }
}


3.How to use ExecutorService with CompletableFuture. Gave a scenario to place an Order with
steps as:- a. initiate payment b. update inventory c. send email to user. Implement the above
using Completable Future


solution


import java.util.concurrent.*;

public class OrderService {

    private static final ExecutorService executor = Executors.newFixedThreadPool(3);

    public static CompletableFuture<String> initiatePayment(String orderId) {
        return CompletableFuture.supplyAsync(() -> {
            simulateDelay("Processing payment");
            return "Payment successful for order: " + orderId;
        }, executor);
    }

    public static CompletableFuture<String> updateInventory(String orderId) {
        return CompletableFuture.supplyAsync(() -> {
            simulateDelay("Updating inventory");
            return "Inventory updated for order: " + orderId;
        }, executor);
    }

    public static CompletableFuture<String> sendEmail(String orderId) {
        return CompletableFuture.supplyAsync(() -> {
            simulateDelay("Sending email");
            return "Email sent to user for order: " + orderId;
        }, executor);
    }

    public static void placeOrder(String orderId) {
        System.out.println("Placing order: " + orderId);

        CompletableFuture<String> paymentFuture = initiatePayment(orderId);
        CompletableFuture<String> inventoryFuture = paymentFuture.thenCompose(paymentResult -> updateInventory(orderId));
        CompletableFuture<String> emailFuture = inventoryFuture.thenCompose(inventoryResult -> sendEmail(orderId));

        emailFuture.thenAccept(finalStatus -> {
            System.out.println("Order complete! " + finalStatus);
        }).exceptionally(ex -> {
            System.out.println("Order failed: " + ex.getMessage());
            return null;
        });

        // Optional: wait for all async tasks to complete before shutting down
        emailFuture.join();
        executor.shutdown();
    }

    private static void simulateDelay(String task) {
        try {
            System.out.println(task + " on thread: " + Thread.currentThread().getName());
            Thread.sleep(1000); // simulate I/O delay
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }

    public static void main(String[] args) {
        placeOrder("ORDER123");
    }
}



4.java program on multi-threading where given a list of integers and no of threads, we need to add 1 to all the no of numbers in the list and return the final list using the given no of threads.

Solution
public class CompletableFutureListIncrementer {

    public static List<Integer> incrementList(List<Integer> list, int threadCount) throws InterruptedException {
        int size = list.size();
        int chunkSize = (int) Math.ceil((double) size / threadCount);

        ExecutorService executor = Executors.newFixedThreadPool(threadCount);
        List<CompletableFuture<Void>> futures = new ArrayList<>();

        for (int i = 0; i < size; i += chunkSize) {
            int start = i;
            int end = Math.min(i + chunkSize, size);

            CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
                for (int j = start; j < end; j++) {
                    list.set(j, list.get(j) + 1);
                }
            }, executor);

            futures.add(future);
        }

        // Wait for all futures to complete
        CompletableFuture.allOf(futures.toArray(new CompletableFuture[0])).join();

        executor.shutdown();
        return list;
    }

    public static void main(String[] args) throws InterruptedException {
        List<Integer> numbers = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10));
        int numberOfThreads = 3;

        List<Integer> result = incrementList(numbers, numberOfThreads);
        System.out.println("Final List: " + result);
    }
}





