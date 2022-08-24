
### Op-Amps

If we manipulate the feedback before its arrival at the inverting terminal, we can modify the values that the op-amp sees. For example, we could cut the feedback values in half via a voltage divider. 

Imagine applying 5V to the input of the following circuit. If the output starts at 0V, the op-amp will compare $(V_+ - V_-) = 5 - 0$ and push current out in order to bring the output positive. In the buffer setup, the comparison $(V_+ - V_-) = 0$ is achieved when the output reaches 5V; however, with the voltage divider, an output of 5V will only cause 2.5V at the input! The output will continue to increase until it reads 5V at the inverting input, or 10V at the output. The "doubling" behavior holds for all practical inputs to this op-amp configuration, and more generally, an op-amp in this configuration is called a "non-inverting" op-amp. The formula for this is:

$$V_{out} = V_{in+} (1 + \frac{R_f}{R_{g}})$$

Notice that if we make $R_f = 0$ or $R_g = \infty$, the formula (and circuit) becomes the same as a buffer, following $V_{out} = V_{in+}$. 


<img src="res/non-inverting-2x.png" height=250 />