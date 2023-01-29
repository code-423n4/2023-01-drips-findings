## Suggestion : distinction of drip receiver and drip sender at dripState 

I recommend keep drip receiver and drip sender states in two different states instead of just having one state . 
a receiver state which keeps required information for receive drips like amtDeltas , nextReceivableCycle , ... and a sender state with information like balance , dripsHash , .... 
that's because when receiving a drip ,  some values of state like balance and dripsHash are not required and vice versa for setdrip  some vales are not changing in sender state ( some values like amtDelta change in receiver state ) . 
so keeping them separate from each other can reduce complexity of codebase and even make it more gas efficient in areas that dripState retrieved . 
Note : when declare a storage variable like state and put dripState in it if you would have more than 4 field accesses, your gas costs would soon exceed the costs of the variant with the memory modifer that's why I believe having two different state with lower size is better than a large state , this will decrease the gas cost .
