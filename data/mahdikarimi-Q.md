## Suggestion : distinction of drip receiver and drip sender at dripState 

I recommend keep drip receiver and drip sender states in two different states instead of just having one state . 
a receiver state which keeps required information for receive drips like amtDeltas , nextReceivableCycle , ... and a sender state with information like balance , dripsHash , .... 
that's because when receiving a drip ,  some values of state like balance and dripsHash are not required and vice versa for setdrip  some vales are not changing in sender state ( some values like amtDelta change in receiver state ) . 
so keeping them separate from each other reduces complexity of codebase .