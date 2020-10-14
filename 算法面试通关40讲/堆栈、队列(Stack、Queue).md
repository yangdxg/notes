栈:先入后出

队列:先入先出

优先队列:正常入,按照优先级出

查找时间复杂度都是O(n),增加、删除时间复杂度都是O(1)

1. 练习232.用栈实现队列

   ```
   class MyQueue {
       
       Stack<Integer> stackIn;
       Stack<Integer> stackOut;
    
       /** Initialize your data structure here. */
       public MyQueue() {
           stackIn = new Stack<>();
           stackOut = new Stack<>();
       }
       
       /** Push element x to the back of queue. */
       public void push(int x) {
           while (!stackOut.empty()){
               stackIn.push(stackOut.pop());
           }
           stackIn.push(x);
       }
       
       /** Removes the element from in front of queue and returns that element. */
       public int pop() {
           while (!stackIn.empty()){
               stackOut.push(stackIn.pop());
           }
           return stackOut.pop();
       }
       
       /** Get the front element. */
       public int peek() {
           while (!stackIn.empty()){
               stackOut.push(stackIn.pop());
           }
           return stackOut.peek();
       }
       
       /** Returns whether the queue is empty. */
       public boolean empty() {
           return stackIn.empty()&&stackOut.empty();
       }
   }
   ```

优先队列的实现机制:

1. 堆(Heap)
2. 二叉搜索树



