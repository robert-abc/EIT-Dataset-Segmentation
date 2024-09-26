</head><body><div class="content"><h2>Contents</h2><div><ul><li><a href="#1">Electrical impedance tomography dataset: Segmentation</a></li><li><a href="#2">Creating a logical image of the tank: original size</a></li><li><a href="#3">Reading original images</a></li><li><a href="#4">Segmentation</a></li><li><a href="#5">Resizing the results</a></li><li><a href="#6">Visualization</a></li></ul></div><h2 id="1">Electrical impedance tomography dataset: Segmentation</h2><pre class="codeinput"><span class="comment">%Targets inside a circular 2D tank</span>
<span class="comment">% For more information:</span>
<span class="comment">% https://fips.fi/open-datasets/eit-datasets/2d-electrical-impedance-tomography-dataset/</span>

<span class="comment">%This is just an approximation.</span>
<span class="comment">%The original photos present parallax, non-uniform illumination and</span>
<span class="comment">%slightly different camera positions for each case</span>
<span class="comment">%This code does not correct this errors.</span>

close <span class="string">all</span>
clear
clc
</pre><h2 id="2">Creating a logical image of the tank: original size</h2><p>Based on <a href="https://matlab.fandom.com/wiki/FAQ#How_do_I_create_a_circle.3F">https://matlab.fandom.com/wiki/FAQ#How_do_I_create_a_circle.3F</a></p><pre class="codeinput"><span class="comment">%Image parameters</span>
imageSizeX = 2000; <span class="comment">%works for square images</span>
imageSizeY = 2000;
[columnsInImage, rowsInImage] = meshgrid(1:imageSizeX, 1:imageSizeY);

<span class="comment">% Circle parameters.</span>
centerX = 1000;
centerY = 1000;
radius = 870; <span class="comment">%This is an approximation</span>

<span class="comment">%Calculating circle points</span>
circlePixels = (rowsInImage - centerY).^2 + (columnsInImage - centerX).^2 &lt;= radius.^2;

<span class="comment">%Converting from logical to double and displaying</span>
vq = double(circlePixels);
</pre><h2 id="3">Reading original images</h2><p>Downloaded from: <a href="https://zenodo.org/records/1203914">https://zenodo.org/records/1203914</a></p><pre class="codeinput"><span class="comment">% Works for the cases without foam: 1-5, 7</span>
data1 = 5;
data2 = 1;
string = <span class="string">"fantom_"</span> + data1 + <span class="string">"_"</span> + data2 + <span class="string">".jpg"</span>;

v2 = imread(string); <span class="comment">%read image</span>
v2 = imresize(v2, [2000, 2000]); <span class="comment">%not all images are 2000 x 2000</span>

v2 = double(im2gray(v2));
v = mat2gray(v2); <span class="comment">%Convert to gray scale</span>

seg_I = vq.*v; <span class="comment">% Set to zero the pixels outside the tank</span>
seg_I = im2double(seg_I);
</pre><h2 id="4">Segmentation</h2><pre class="codeinput">thresh = graythresh(seg_I); <span class="comment">%Otsu's method for binary segmentation</span>
seg_I = double(imbinarize(seg_I,thresh));

seg_I=medfilt2(seg_I,[5,5]); <span class="comment">%median filter to remove small artifacts</span>

seg_fill = imfill(seg_I, <span class="string">'holes'</span>); <span class="comment">%(morphological filtering) Opening, because the conductive targets are rings</span>

<span class="comment">%Close and opening to remove small artifacts</span>
se = strel(<span class="string">'disk'</span>,15);
seg_I_closed = imclose(seg_fill,se); <span class="comment">%</span>
J = imopen(seg_I_closed,se);

<span class="comment">% Second Otsu's segmentation</span>
thresh2 = graythresh(J);

J = imbinarize(J,thresh2);

<span class="comment">% J(J&gt;thresh2) = 1;</span>
<span class="comment">% J(J&lt;thresh2) = 0;</span>
<span class="comment">% J(J==thresh2) = 1;</span>
</pre><h2 id="5">Resizing the results</h2><pre class="codeinput"><span class="comment">%New image parameters</span>
imageSizeX2 = 64;
imageSizeY2 = 64;

segmented3 = vq + J; <span class="comment">%tank and targets masks</span>
segmented3( ~any(segmented3,2), : ) = [];  <span class="comment">%Remove the rows with zeros</span>
segmented3( :, ~any(segmented3,1) ) = [];  <span class="comment">%Remove the columns with zeros</span>
segmented3(segmented3&gt;1) = 0;

<span class="comment">%Resize with nearest neightbors method</span>
segmented_resized = imresize(segmented3, [imageSizeX2, imageSizeY2], <span class="string">'nearest'</span>);
</pre><h2 id="6">Visualization</h2><pre class="codeinput">figure
subplot(2,2,1)
imshow(v2 - J,[])
axis <span class="string">square</span>
title(<span class="string">'Original image: 2000 x 2000'</span>)

subplot(2,2,2)
imshow(vq - J,[])
axis <span class="string">square</span>
title(<span class="string">'Segmented: 2000 x 2000'</span>)

subplot(2,2,3)
imshow(segmented3)
axis <span class="string">square</span>
title(<span class="string">'Selected area'</span>)

subplot(2,2,4)
imshow(segmented_resized)
axis <span class="string">square</span>
title(<span class="string">'Segmented: 64 x 64'</span>)
</pre><img vspace="5" hspace="5" src="mask_seg_01.png" alt=""> 
