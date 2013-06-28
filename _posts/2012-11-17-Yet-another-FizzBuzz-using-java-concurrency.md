---
layout: post
title: Yet another FizzBuzz using java concurrency
---

*FizzBuzz* the famous interview question to test the programmer skills ;).
Recently got caught in this situation. What is this FizzBuzz?
According to wiki [http://en.wikipedia.org/wiki/Bizz_buzz](BizzBuzz):

"Bizz buzz (also known as fizz buzz, or simply buzz) is a group word game frequently encountered as a drinking game.
Players take turns to count incrementally, replacing any number divisible by three with the word "fizz", and any number divisible by five with the word "buzz". And numbers divisible by both become Fizzbuzz"

Sample output is : 1, 2, Fizz, 4, Buzz, Fizz, 7, 8, Fizz, Buzz, 11, Bizz, 13, 14, FizzBuzz, 16, 17, Fizz, 19, Buzz..

Therefore, I wrote the below given code to fulfil the requirement:

    public class QuickFizzBuzz {
        public static void main(String a[]){
            for(int i=1; i<=100; i++){
                if (i % 3 == 0 && i % 5 == 0)
                    System.out.println("FIZZBUZZ");
                else if (i % 3 == 0)
                    System.out.println("FIZZ");
                else if (i % 5 == 0)
                    System.out.println("BUZZ");
                else
                    System.out.println("" +i);
            }
        }
    }

But the interviewer is not happy.
So, how do we over engineer it.
Lets use concurrency to solve the fizzbuzz:

FizzBuzzProcess.java:

    import java.util.HashMap;
    import java.util.concurrent.Callable;

    public class FizzBuzzProcess implements Callable<HashMap<Integer, String>>{

        private int start;
        private int end;
        
        public FizzBuzzProcess(int start, int end){
            this.start = start;
            this.end = end;
        }
    
        @Override
        public HashMap<Integer, String> call() throws Exception {
            HashMap<Integer, String> results = new HashMap<Integer, String>(end-start);
            
            for (int i=start; i<=end; i++){
                if (i % 3 == 0 && i % 5 == 0)
                    results.put(i, "FIZZBUZZ");
                else if (i % 3 == 0)
                    results.put(i, "FIZZ");
                else if (i % 5 == 0)
                    results.put(i, "BUZZ");
                else
                    results.put(i, "" +i);
            }
            return results;
        }
    }

ValueComparator.java:

    import java.util.Comparator;
    import java.util.Map;

    public class ValueComparator implements Comparator<Integer> {

        Map<Integer, String> base;
        public ValueComparator(Map<Integer, String> base) {
            this.base = base;
        }

        public int compare(Integer a, Integer b) {
            if (a <= b) {
                return -1;
            }
            else {
                return 1;
            } // returning 0 would merge keys
        }
    }

FizzBuzzMain.java:

    import java.util.HashMap;
    import java.util.HashSet;
    import java.util.List;
    import java.util.Map;
    import java.util.TreeMap;
    import java.util.concurrent.ExecutorService;
    import java.util.concurrent.Executors;
    import java.util.concurrent.Future;

    public class FizzBuzzMain {

        public HashMap<Integer, String> getResult(int size){
            HashMap<Integer, String> results = new HashMap<Integer, String>(size);
        
            HashSet<FizzBuzzProcess> processes = new HashSet<FizzBuzzProcess>();
            processes.add( new FizzBuzzProcess(1, size/2));
            processes.add( new FizzBuzzProcess(size/2 + 1, size));
        
            ExecutorService service = Executors.newFixedThreadPool(2);
            List<Future<HashMap<Integer, String>>> futures = null;
        
            try{
                futures = service.invokeAll(processes);
            
                for (Future<HashMap<Integer, String>> f: futures){
                    results.putAll(f.get());
                }
            }catch(InterruptedException e){
                e.printStackTrace();
            }
            catch(Exception e){
                e.printStackTrace();
            }
            finally{
                if (!service.isShutdown()){
                    service.shutdown();
                }
            }
            return results;
        }
    
        public void print(HashMap<Integer, String> output){
            ValueComparator vc = new ValueComparator(output);
            TreeMap<Integer, String> sortedMap = new TreeMap<Integer, String>(vc);
            sortedMap.putAll(output);
            for(Map.Entry<Integer, String> entry: sortedMap.entrySet()){
                System.out.println(entry.getValue());
            }
        }
    
        public static final void main(String a[]){
            FizzBuzzMain f = new FizzBuzzMain();
            f.print( f.getResult(100));
        }
    }
