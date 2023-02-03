
# Gas Optimizations Report

This report focuses on Drips contest, in context of various improvements that can be made in terms of gas cost.

Some of the opportunities identified for improving gas efficiency throughout the codebase of Drips Protocol are categorised into 07 main areas; with further multiple instances in each of the category.


# Summary

[G-01] State variable values are by default internal (11 Instance)
[G-02] Immutable has more gas efficiency than constant (08 Instances)
[G-03] Multiple mappings can be combined into a single one (02 Instances)
[G-04] Uint/Int lower than 32 bytes incurs overhead (125 Instances)
[G-05] Wastage of deployed gas when return is not present for returns function (22 Instances)
[G-06] 0perator assignment is more gas efficient than compound assignment (21 Instances)
[G-07] Functions with public visibility, if not called within the contract needed to be changed external (17 Instances) 
# [G-01] State variable values are by default internal (11 Instance)

When visibility for a state variable is not specified, the default value is already internal.

Declaring visibility of the variable internal consumes extra gas which is not needed.

Link to the Code:
1.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L82
2.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L84
3.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L106
4.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L108
5.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L110
6.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L112
7.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L117
8.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L20
9.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L22
10.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L25
11.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L88


# [G-02] Immutable has more gas efficiency than constant (08 Instances)

Using immutable instead of constant, save more gas due to avoiding storage access for state variables.

Variable values are set through constructor when using immutable, which also eliminates the need of assigning initial values to state variable making them more efficient in terms of gas cost

Link to the Code:
1.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L56
2.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L58
3.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L60
4.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L62
5.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L64
6.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L67
7.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L70
8.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L82


# [G-03] Multiple mappings can be combine into a single one (02 Instances)

When multiple mappings are used in same function, it’s better to combined them into a single mapping of an address struct.

Combined mapping reduces storage slot per mapping and also are cheaper in terms of associated Stack operations calculation carried out.

Link to the Code:
1.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L88
2.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L90


# [G-04] Uint/Int lower than 32 bytes consumes incurs overhead (125 Instances)

Contract gas usage increases as EVM standard operation are of 32 bytes. If any element is smaller than 32 bytes (i.e.; 256 bits) it will cause EVM to consume more gas which can be around 12 gas depending on size for reducing the size to given output like uint8.


Link to the Code:

1.	https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L15
2.	https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L17
3.	https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L28
4.	https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L25
5.	https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L30
6.	https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L41
7.	https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L60
8.	https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L76
9.	https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L122
10.	https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L170
11.	https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L25
12.	https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L32
13.	https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L133
14.	https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L186
15.	https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L285
16.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L11
17.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L33
18.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L42
19.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L49
20.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L57
21.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L71
22.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L86
23.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L98
24.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L106
25.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L117
26.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L145
27.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L176
28.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L185
29.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L199
30.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L228
31.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L233
32.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L58
33.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L64
34.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L77
35.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L84
36.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L97
37.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L109
38.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L119
39.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L134
40.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L145
41.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L152
42.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L160
43.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L169
44.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L216
45.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L238
46.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L276
47.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L303
48.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L328
49.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L358
50.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L372
51.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L390
52.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L409
53.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L438
54.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L439
55.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L440
56.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L464
57.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L465
58.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L517
59.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L518
60.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L519
61.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L521
62.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L533
63.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L535
64.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L536
65.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L560
66.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L561
67.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L629
68.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L635
69.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L643
70.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L28
71.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L30
72.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L60
73.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L73
74.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L78
75.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L83
76.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L88
77.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L112
78.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L117
79.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L135
80.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L136
81.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L153
82.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L169
83.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L185
84.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L189
85.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L191
86.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L193
87.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L195
88.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L197
89.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L203
90.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L208
91.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L210
92.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L219
93.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L233
94.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L270
95.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L303
96.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L317
97.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L348
98.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L402
99.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L475
100.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L514
101.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L515
102.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L516
103.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L538
104.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L561
105.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L619
106.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L685
107.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L855
108.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L856
109.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L875
110.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L876
111.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L878
112.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L912
113.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L914
114.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L923
115.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L924
116.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L936
117.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L944
118.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L949
119.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L970
120.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L991
121.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1020
122.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1036
123.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1120
124.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1128
125.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1134


# [G-05] Wastage of deployed gas when return is not present for returns function (22 Instances)

Wastage of gas during deployment; when return is absent for named variable when function returns.

Link to the Code:
1.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L235
2.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L273
3.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L317
4.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L348
5.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L401
6.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L448
7.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L561
8.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L619
9.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L219
10.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L276
11.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L303
12.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L390
13.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L519
14.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L120
15.	https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L65
16.	https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L82
17.	https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L91
18.	https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L113
19.	https://github.com/code-423n4/2023-01-drips/blob/main/src/NFTDriver.sol#L196
20.	https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L60
21.	https://github.com/code-423n4/2023-01-drips/blob/main/src/AddressDriver.sol#L131
22.	https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L56


# [G-06] 0perator assignment is more gas efficient than compound assignment (21 Instances)

Compound assignment operators (+= / -=) are more expensive in terms of gas consumption and needs to be avoided.

Operator assignments (a = a + b / a - b) are preferable in terms of gas optimization.

Link to the Code:
1.	https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L62
2.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L253
3.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L284
4.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L288
5.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L289
6.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L290
7.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L429
8.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L495
9.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L572
10.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L755
11.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1053
12.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Drips.sol#L1054
13.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L632
14.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L636
15.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L99
16.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L128
17.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L159
18.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L162
19.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L167
20.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L168
21.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Splits.sol#L235


# [G-07] Functions with public visibility, if not called within the contract needed to be changed external (17 Instances)

Functions with public visibility, if not called within the contract needed to be changed external.

Link to the Code:
1.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L134
2.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L152
3.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L160
4.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L169
5.	https://github.com/code-423n4/2023-01-drips/blob/main/src/DripsHub.sol#L608
6.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L90
7.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L97
8.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L112
9.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L122
10.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Managed.sol#L128
11.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L106
12.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L114
13.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L124
14.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L132
15.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L144
16.	https://github.com/code-423n4/2023-01-drips/blob/main/src/Caller.sol#L193
17.	https://github.com/code-423n4/2023-01-drips/blob/main/src/ImmutableSplitsDriver.sol#L53