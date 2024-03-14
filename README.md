## Radar Target Generation and Detection 
In this project, the following tasks should be done:
- Configure the FMCW waveform based on the system requirements.
- Define the range and velocity of target and simulate its displacement.
- For the same simulation loop process the transmit and receive signal to determine the beat signal
- Perform Range FFT on the received signal to determine the Range
- Towards the end, perform the CFAR processing on the output of 2nd FFT to display the target.


### Radar System Requirements
|Frequency | 77 Ghz|
:----:|:----:
|Range Resolution | 1 m|
|Max Range | 200 m|
|Max velocity | 70 m/s|
|Velocity resolution| m m/s|

### Range FFT (1st FFT)
#### FFT Operation
- Implement the 1D FFT on the Mixed Signal
- Reshape the vector into Nr*Nd array.
- Run the FFT on the beat signal along the range bins dimension (Nr)
- Normalize the FFT output with length, L = Bsweep * Tchirp.
- Take the absolute value of that output.
- Keep one half of the signal
- Plot the output
- There should be a peak at the initial position of the target.


### 2D CFAR
- Determine the number of Training cells for each dimension. Similarly, pick the number of guard cells. <br />
```
%Select the number of Training Cells in both the dimensions.
Tr=10;
Td=8;
%Select the number of Guard Cells in both dimensions around the Cell under 
Gr=4;
Gd=4;
% offset the threshold by SNR value in dB
offset=6;
```
- Slide the cell under test across the complete matrix. Make sure the CUT has margin for Training and Guard cells from the edges.
- For every iteration sum the signal level within all the training cells. To sum convert the value from logarithmic to linear using db2pow function.
- Average the summed values for all of the training cells used. After averaging convert it back to logarithmic using pow2db.
- Further add the offset to it to determine the threshold.
- Next, compare the signal under CUT against this threshold.
- If the CUT level > threshold assign it a value of 1, else equate it to 0.
- The process above will generate a thresholded block, which is smaller than the Range Doppler Map as the CUTs cannot be located at the edges of the matrix due to the presence of Target and Guard cells. Hence, those cells will not be thresholded. To keep the map size same as it was before CFAR, equate all the non-thresholded cells to 0.
```
RDM_copy = RDM / max(RDM(:));
for i = Tr+Gr+1 : (Nr/2)-(Tr+Gr)
    for j = Td+Gd+1 : Nd-(Gd+Td)
        noise_level = zeros(1,1);
    
   % Use RDM[x,y] as the matrix from the output of 2D FFT for implementing
   % CFAR
        for p = i-(Tr+Gr) : i+Tr+Gr
            for q = j-(Td+Gd) : j+Td+Gd 
                if (abs(i-p)>Gr || abs(j-q)>Gd)
                    noise_level = noise_level + db2pow(RDM_copy(p,q));
                end
            end
        end
        threshold = pow2db(noise_level/(2*(Td+Gd+1)*2*(Tr+Gr+1) - (Gr*Gd) - 1));
        threshold = threshold + offset;

        if(RDM_copy(i,j) > threshold)
            RDM(i,j) = 1;
        else
            RDM(i,j) = 0;
        end
    end
end
```