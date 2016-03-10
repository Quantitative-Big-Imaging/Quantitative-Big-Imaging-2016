# Exercise 3: Segmentation



## Loading workflows
Download the workflows [here](https://github.com/kmader/Quantitative-Big-Imaging-2016/blob/master/Exercises/03-files/Workflows.zip?raw=true)

Many of these workflows are fairly complicated and would be time consuming to reproduce, follow the instructions [here](https://github.com/kmader/Quantitative-Big-Imaging-2016/wiki/KNIME-Setup#loading-workflows) for how to import a workflow from the zip files on this site



## Problems!
If you load a workflow and get an error message, click on the details button. If it says 'Node ... not available' it means you need to update your 'Image Processing Extensions' follow the instructions below to perform this update: [instructions](https://github.com/kmader/Quantitative-Big-Imaging-2016/wiki/KNIME-Setup#updating-to-the-latest-image-processing-extensions)

- If you cannot find the 'Salt and Pepper' node, this also means you are not using the latest Image Processing Extensions so update it as described above

## Getting Started
- Steps are shown in normal text, comments are shown in _italics_.

- Knime Basics: [here](https://github.com/kmader/Quantitative-Big-Imaging-2016/wiki/KNIME-Setup)

- Use workflow variables: [here](https://github.com/kmader/Quantitative-Big-Imaging-2016/wiki/KNIME-Setup#workflow-variables)

## Bone-Segmentation

### Simple

 Here we have the simple task of segmenting the calcified bone tissue from the air and background in the image
 
 - Apply a threshold and then a morphological operation to segment the calcified tissue well
 - Calculate the porosity (air) volume fraction for each threshold (_hint_: Image Features)
 - Improve the images using morphological operations and recalculate the porosity (_hint_: Morphological Image Operations)
 - Fill in all the holes and estimate the bone volume to total volume an important diagnostic metric in biomechanics (_hint_: Fill Holes)
 - Total volume is __not__ the total volume of image but rather the size of the bone inside the image
 - Add a filtering block, which works best on these data?


### ManyThresh

Here we try a number of different thresholds to identify the bone better.

1. Adjust the list of thresholds to cover a reasonable range
1. Adjust the morphological operation to it produces the best result
1. Add many more steps around the ideal value how does the curve look?

## Cell-Segmentation

The cell segmentation takes the process one step further and tries to identify the cells by looking for holes inside the bone tissue. 

1. Adjust the threshold to the best value as determined before
1. Adjust the iterations for the morphological operation in the _Create Bone_ to fill the bone in better
1. Adjust the settings in _Create Cells_ to better segment the cells

***

1. Check the results using the _Interactive Segmentation Viewer_
1. Generate the report as a PDF
1. Make a plot of the X position against the maximum intensity, is there any trend? What might this be from?
1. _Optional_: Add 'Haralick' feature analysis to the output, plot some of the parameters against position


Additional Examples
===
KNIME is not the tool for everything, and we won't spend too much time on the others but 
 - FIJI (ImageJ with all the plugins already installed): Visualization, Preprocessing, Segmentation, Morphometrics [http://www.fiji.sc]
 - Paraview
 - ImageJ Plugins (Java and Python)


## Looking at the Fossil Data

#### From [Paleontology Paper](http://onlinelibrary.wiley.com/doi/10.1111/j.1475-4983.2010.01006.x/abstract)
#### Data courtesy of Philip Donaghue and John Cunningham
- [Download Here](http://people.ee.ethz.ch/~maderk/data.zip)

### Intenstine
- Folder: Gut-PhilElvCropped
- Load into FIJI
- Rescale Brightness and Contrast
- Visualize
- Perform Threshold
- Visualize


***

[Video Instructions](http://people.ee.ethz.ch/~maderk/videos/DoesMyFossilHaveTeeth.swf)

### Teeth 
#### Folder: Teeth-HiResPhilGix
1. Load into FIJI
1. Adjust brightness and contrast
1. Visualize
1. Perform Threshold
1. Visualize
1. Count the number of teeth



## Advanced: Paraview

- What is ParaView
 - parallel, distributed image processing and visualization toolbox written in C++
 - open source, extendable API
 - pipeline style analyses


### Loading Data Into Paraview
[Video Instructions](http://people.ee.ethz.ch/~maderk/videos/ParaviewLoadThreshold.swf)
Additional more detailed
[Web Instructions](http://wiki.rac.manchester.ac.uk/community/ParaView/Tips/LoadImageStack)

1. Load Data in
1. Apply a threshold
1. Visualize the segmentation
1. Crop the image
1. Experiment with other features


### Developing ImageJ2 / KNIME Plugins

- Java classes, but with a well-developed API and GUI wrapped around it
- Can be imported and run in KNIME or ImageJ2/FIJI
- Powerful flexible framework behind: [ImgLib2](http://imglib2.net/), [examples](http://fiji.sc/ImgLib2_Examples)

```{java}
@Plugin(headless = true, type = Command.class)
public class DemoPlugin<T extends RealType<T>> 
  implements Command {
  @Override
  public void run() {
    // Code for your plugin
  }
}
```

### ImageJ2 Average Value

```{java}
public void run() {
        final RealSum realSum = new RealSum();
        long count = 0;
 
        for ( final T type : input ) {
            realSum.add( type.getRealDouble() );
            count++;
        }
        return realSum.getSum() / count;
}

```



### Old Style ImageJ Plugins (Most existing ones)

- Can be "recorded" like Macro's to call other functions or plugins
- Examples 
- [Filters in 3D](http://imagejdocu.tudor.lu/doku.php?id=plugin:filter:3d_filters:start)
- [Fast 3D Filters](http://imagejdocu.tudor.lu/doku.php?id=plugin:filter:3d_filters_with_jni:start) - native libraries allow for really fast implementations

```
public class My_Plugin implements PlugIn {
  public void run(String arg) {
		// Code for your plugin
	}
}
```

ImageJ / FIJI Plugins: Average Value
========================================================
- __Plugins__ $\rightarrow$ New $\rightarrow$ Plugin, [Download](https://gist.github.com/kmader/9369975)

```
public void run(String arg) {
	ImagePlus imp = IJ.getImage();
	ImageStack is = imp.getStack();
	int n_slices = is.getSize();
	for(int sliceIndex=1;sliceIndex<=n_slices;sliceIndex++) {
	  double sumVal=0;
		long countPixel=0;
		ImageProcessor curSlice=is.getProcessor(sliceIndex);
		float[][] sliceArray=curSlice.getFloatArray();
		for(int x=0;x<sliceArray.length;x++) {
			for(int y=0;y<sliceArray[x].length;y++) {
				sumVal+=sliceArray[x][y];
				countPixel+=1;
			}	
		}
		IJ.log("Slice:"+sliceIndex+", Mean:"+sumVal/countPixel);
		}
	}
```


FIJI Python Plugins: Average Value
========================================================
- __New__ $\rightarrow$ Script, then __Language__ $\rightarrow$ Python, [Download](https://gist.github.com/kmader/9369798)

```
image = WindowManager.getCurrentImage()
stack = image.getStack() # get the stack within the ImagePlus
n_slices = stack.getSize() # get the number of slices
# iterate over every slice
for sliceIndex in range(1,n_slices+1):
  curSlice=stack.getProcessor(sliceIndex)
  sumVal=0;countPixel=0
  # iterate over every pixel in the slice
  for cPixel in curSlice.toFloatArrays()[0]: 
    sumVal+=cPixel
    countPixel+=1
  IJ.log("Slice:"+str(sliceIndex)+", Mean:"+str(sumVal/countPixel))
```