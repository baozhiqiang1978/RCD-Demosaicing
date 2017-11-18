# RCD-Demosaicing
A Bayer CFA demosaicing routine aimed to reduce pixel overshooting.

## Brief overview

The Ratio Corrected Demosaicing algorithm intends to smooth the colour correction errors that are common in many other interpolation methods. To achieve this goal, the algorithm operates at two different levels:
* It uses a novel directional discrimination statistic which is invariant to chromatic aberrations.
* It estimates the pixel value applying a smoothing ratio to the spatial difference correction in a low-pass filter domain instead of in the more usual colour difference in the CFA domain.

The algorithm comprises four parts:

### 1. Local directional discrimination
The algorithm calculates the relative 1D local gradients directly on the Bayer data for the horizonal, vertical, diagonal M and diagonal P directions.

It then refines the calculation trying to infere more discriminational strength from the neighbourhood wherever possible.

### 2. Low-pass filter
Taking advantadge of the structure of the Bayer CFA, a very simple and stable low-pass filter is used to smooth the colour intensity differences between the red and the blue channels.

    1 2 1
    2 4 2 * CFA
    1 2 1

This convolution results in a grey scale image which is softer than the input but is almost artifact free.

### 3. Green channel interpolation
Using the data from the steps #1 and #2, the green channel is updated using a pixel estimation based on ratios. Instead of the widespread Hamilton-Adams interpolation, the proposed ratio-corrected interpolation applies the colour correction taking into account the context of the intensity difference between spatially separated pixels. This way, in hard edges colour overshooting issues are under control.

Example for the East pixel value estimation:

    Hamilton-Adams  => Gest(0) = G(1) + ( R(0) - R(2) ) / 2
    Ratio-corrected => Gest(0) = G(1) * ( 1 + ( R(0) - R(2) ) / ( R(0) + R(2) ) )

To further smooth the output, the low-pass filter from step #2 is introduced in the formula:

    Ratio-corrected LPF => Gest(0) = G(1) * ( 1 + ( LPF(0) - LPF(2) ) / ( LPF(0) + LPF(2) ) )

### 4. Red and Blue channels interpolation
The routine used for interpolating the chrominance is fairly simple yet surprisingly effective. Using the full green channel from step #3, the red and blue channels are updated with the local colour differences.

The gradients are empirically optimized.

## Usage

The algorithm can be plugged to dcraw. It expects two buffers to be declared prior to its include command:

    float (*cfa), containing the CFA values in a [0-1] floating point range
    float (*rgb)[3], containing the int (*image)[4] values in a [0-1] floating point range

Both can be filled in a single pass to all the image pixels:

    cfa[indx] = rgb[indx][FC(row,col)] = (float) image[indx][FC(row,col)] / 65535.f;

The algorithm outputs its results by modifying the buffer "rgb", which later should be returned to the usual form of image[4] dcraw uses:

    FORCC image[row*width+col][c] = (ushort) CLIP( 65535.f*rgb[row*width+col][c] );
