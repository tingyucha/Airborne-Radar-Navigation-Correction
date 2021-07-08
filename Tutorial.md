# Tutorial

### 1. Install the navigation correction packpage 

* Install the package following the [instructions.](https://github.com/tingyucha/Airborne-Radar-Navigation-Correction/blob/main/README.md) 
* Install soloii

### 2. Select a flight leg and set up the directories

Select a ~10-minute period during your flight where the aircraft is flying in a straight line and at a constant altitude. Ideally, this 10-minute period is 

(1) in clear weather, where the ground shows up clearly in the reflectivity field and is not masked by other echoes; and 

(2) over land, where we know that the ground isn’t moving. But, some light rain and/or while the aircraft is over the ocean is still acceptable. 

In hurricane reconnaissance flights, outbound or inbound legs are typically good. You will need to know the exact number of sweep files in that 10-minute period.

Below is an example of applying the navigation correction on a NOAA43 flight into Hurricane Ophelia (2005) on September 11, 2005.
* Set up your navigation correction directory as follows. Let’s call this directory **navcorr**:

![image](https://user-images.githubusercontent.com/25255478/124947659-442cab80-dfcd-11eb-9d8d-8bfae954a4c7.png)

File is in green color, folder/directory is in blue color, and executable is in red color.

Following the commands below to convert the ar2v to sweep files.
* Convert the levelII data to sweep files

```terminal
RadxConvert -dorade -f *.ar2v
```

* Store the sweep files from the 10-minute period in the swp folder.

### 3. Initial Navigation Correction

In this example, radar reflectivity is DZ, spectrum width is SPEC_WDT, and Doppler velocity is VE.

![image](https://user-images.githubusercontent.com/25255478/124950634-d59d1d00-dfcf-11eb-8d77-2cf908c0265c.png)

* Copying these fields to a new field with their common names in soloii. Depending on how your data was unpacked, you may not need to do this step. 

```terminal
copy VE to VEL; copy DZ to DBZ; copy SPEC_WDT to WIDTH; copy WIDTH to SW
```

* Create a field called NCP and assign a value of 1 to it.

```terminal
copy DBZ to NCP; assign-value 1 to NCP
```

* Applying the initial navigation correction to the sweep files. Make sure to apply the corrections to all sweep files by clicking on **First Sweep** and **Last Sweep**, and also to both the fore sweeps and the aft sweeps.

```terminal
copy VEL to VG
remove-aircraft-motion in VG
copy VG to VR
```

![image](https://user-images.githubusercontent.com/25255478/124951928-0893e080-dfd1-11eb-8509-8bc0c9058e02.png)


### 4. Main Navigation Correction

In the main navigation correction steps, you would have to run the software on the whole set of sweep files that you have, but apply the correction factor to the fore sweeps and aft sweeps individually. Rather than moving the files back and forth between the swp folder and the fore/aft folders, you can just create a symlink in the fore/aft folders that point toward the respective files in the swp folder.

Now, you want to convert all the sweep files in the swp folder into cfradial in a folder called cfradial in the main directory:

```terminal
RadxConvert -f /navcorr/swp/swp.1050911* -outdir /navcorr/cfradial
```

Radx will convert the sweep files into cfradial files in /navcorr/cfradial/20050911. Go to the directory with the cfradial files:

```terminal
cd /navcorr/cfradial/20050911/
```

Now, create a list file of all the generated cfradial files called “filelist”

```terminal
ls * > filelist
```

You will need to insert the number of sweep/cfradial files that you have in the top line, like this:


![image](https://user-images.githubusercontent.com/25255478/124952933-e484cf00-dfd1-11eb-80dc-cef13de1c097.png)

Close filelist and run:

```terminal
./navcorr/readnetcdf_DBZ_VR	filelist
```

Your terminal should start spitting out things like this:

![image](https://user-images.githubusercontent.com/25255478/124953060-08481500-dfd2-11eb-873f-1ac52e1f2902.png)

After it is done running, a bunch of text files will be created in the same directory as the cfradial files.

Go back to the main directory: 

```terminal
cd /navcorr
```

Move the folder containing the cfradial files in the **/navcorr** directory (or anywhere but the cfradial folder). In this example, I moved it into the /navcorr directory. Now, move the generated text files into the cfradial folder:

```terminal
mv 20050911/*.txt cfradial
```

This is where the navigation correction software will look for the generated text files (second line in DATA_cns_run). Of course, you can change this by changing the second line in DATA_cns_run. You can also change the name of the output directory in the DATA_cns_run file instead of “corrections”

Now, edit the fields in the DATA_cns_run text file. Highlighted are the things that you would need to change according to the details of your own flight:

![image](https://user-images.githubusercontent.com/25255478/124953287-3f1e2b00-dfd2-11eb-8f58-20da9a981de5.png)

Generate correction factors file:

```terminal
./cns_eldo_cai	DATA_cns_run
```

The program should run and list a bunch of things. cfac files should be in **corrections**.

Copy cfac.aft/cfac.fore into archive as cfac.aft1/cfac.fore1


More importantly, copy cfac.aft into **/navcorr/aft** and cfac.fore into **/navcorr/fore**

Go into either the fore or aft directory. In this case, I went into the fore directory. Export the cfac file before opening solo. Note that the sweep files have designation for fore sweeps and aft sweeps. In this example, I exported the cfac file cfac.fore to be read-in when the fore sweeps TF43P3 were accessed.

![image](https://user-images.githubusercontent.com/25255478/124953584-860c2080-dfd2-11eb-897e-57e37850a789.png)

To ensure that the cfac file was read-in, when you open solo, you should have this notification:

![image](https://user-images.githubusercontent.com/25255478/124953638-8f958880-dfd2-11eb-9501-f5175f0e7928.png)

Now, run these commands in the editor. Remember to apply the commands to all the sweeps by clicking “First Sweep” and “Last Sweep.”

![image](https://user-images.githubusercontent.com/25255478/124953690-99b78700-dfd2-11eb-9579-bcbb408436d2.png)


It should say “Finished!” in the terminal. After this, whatever you do on solo, it will crash. Best to close it either way since you have nothing else to do with solo.

Now go to the other directory and repeat the same thing. Since I was in the fore directory last, we’ll go into the aft directory.

Again, when exporting cfac files, make sure you have the correct file designation. Since I’m working on the aft files, the file name should be **TA43P3**

![image](https://user-images.githubusercontent.com/25255478/124953751-a5a34900-dfd2-11eb-9467-6392901c2c82.png)


If the cfac files were not properly exported, solo will just open without the “Opening cfac file” notification.

Repeat the same commands in solo as I did in the fore sweeps. Solo will crash. If you try to open solo to view other things, and the cfac file is not in the same directory, it won’t open because it won’t be able to find the cfac file. You would need to unset the cfac file using the command:

```terminal
unset CFAC_FILES
```

Now remove everything in the corrections folder (you’ve saved the cfac files in “archive”), remove the folders containing the generated text files and cfradial files.


```terminal
rm corrections/*
rm -r cfradial
rm -r 20050911
```

Repeat the whole process from converting the sweep files into cfradial.

```terminal
cd /navcorr/swp
RadxConvert -f swp* -outdir ../cfradial
```
If the navigation correction is not retained in the converted cfradial files, you may want to try the command below to keep the existing navigation correction:

```terminal
•	RadxConvert -apply_georefs -primary_axis y_prime -radar_num 2 -f ./swp*
```

Once you’ve run the navigation correction again, you should see the correction values decrease relative to the first navigation correction that you did.

![image](https://user-images.githubusercontent.com/25255478/124954105-fc108780-dfd2-11eb-91ad-599b7b678917.png)


You typically want to run the navigation correction to the point where the correction values are close to zero. What you do beyond this point is to add the correction values in the second round of navigation correction to the correction values in the first round of navigation correction, export the “new” cfac file to solo and repeat the commands in solo. Then run the navigation correction again and you would see that the correction values will approach zero more than in the second round of navigation correction. The cfac files that you exported that resulted in correction factors closest to zero (before you stop) is the one you want to apply to the rest of the sweeps.


