> Number Plate Detection and Recognization using openCV and EasyOCR

## Project Done under the guidance of Dr. Malaya Dutta Borah mam by:

### Keerthi Satya Sai Sundar
### Karri Sai Venkata Reddy
### Karthik Shanam
### Kadimi Varun Chandra Sai




Steps involved in License Plate Recognition
1. License Plate Detection: The first step is to detect the License plate from the car. We will use the contour option in OpenCV to detect for rectangular objects to find the number plate. The accuracy can be improved if we know the exact size, color and approximate location of the number plate. Normally the detection algorithm is trained based on the position of camera and type of number plate used in that particular country. This gets trickier if the image does not even have a car, in this case we will an additional step to detect the car and then the license plate.

2. Character Segmentation: Once we have detected the License Plate we have to crop it out and save it as a new image. Again this can be done easily using OpenCV.

3. Character Recognition: Now, the new image that we obtained in the previous step is sure to have some characters (Numbers/Alphabets) written on it. So, we can perform OCR (Optical Character Recognition) on it to detect the number

1. License Plate Detection
Let’s take a sample image of a car and start with detecting the License Plate on that car. We will then use the same image for Character Segmentation and Character Recognition as well.


Step 1: Resize the image to the required size and then grayscale it.

The code for the same is given below

#img = cv2.imread('image.jpg')
#gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
#plt.imshow(cv2.cvtColor(gray, cv2.COLOR_BGR2RGB))

Resizing will help us to avoid any problems with bigger resolution images, make sure the number plate still remains in the frame after resizing. Gray scaling is common in all image processing steps. This speeds up other following process sine we no longer have to deal with the color details when processing an image. The image would be transformed something like this when this step is done


Step 2: Every image will have useful and useless information, in this case for us only the license plate is the useful information the rest are pretty much useless for our program. This useless information is called noise. Normally using a bilateral filter (Blurring) will remove the unwanted details from an image. The code for the same is blurred

bfilter = cv2.bilateralFilter(gray, 11, 17, 17) #Noise reduction
edged = cv2.Canny(bfilter, 30, 200) #Edge detection
plt.imshow(cv2.cvtColor(edged, cv2.COLOR_BGR2RGB))


Syntax is destination_image = cv2.bilateralFilter(source_image, diameter of pixel, sigmaColor, sigmaSpace). You can increase the sigma color and sigma space from 15 to higher values to blur out more background information, but be careful that the useful part does not get blurred. The output image is shown below, as you can see the background details (tree and building) are blurred in this image. This way we can avoid the program from concentrating on these regions later.

The next step is interesting where we perform edge detection. There are many ways to do it, the most easy and popular way is to use the canny edge method from OpenCV. The line to do the same is shown below

The syntax will be destination_image = cv2.Canny(source_image, thresholdValue 1, thresholdValue 2). The Threshold Vale 1 and Threshold Value 2 are the minimum and maximum threshold values. Only the edges that have an intensity gradient more than the minimum threshold value and less than the maximum threshold value will be displayed. The resulting image is shown below


Step 3: Now we can start Find Contours and Apply Mask

keypoints = cv2.findContours(edged.copy(), cv2.RETR_TREE,
cv2.CHAIN_APPROX_SIMPLE)
contours = imutils.grab_contours(keypoints)
contours = sorted(contours, key=cv2.contourArea, reverse=True)[:10]


Once the counters have been detected we sort them from big to small and consider only the first 10 results ignoring the others. In our image the counter could be anything that has a closed surface but of all the obtained results the license plate number will also be there since it is also a closed surface.

To filter the license plate image among the obtained results, we will loop though all the results and check which has a rectangle shape contour with four sides and closed figure. Since a license plate would definitely be a rectangle four sided figure.

location = None
for contour in contours:
approx = cv2.approxPolyDP(contour, 10, True)
if len(approx) == 4:
location = approx
break


Once we have found the right counter we save it in a variable called screenCnt and then draw a rectangle box around it to make sure we have detected the license plate correctly.

Now that we know where the number plate is, the remaining information is pretty much useless for us. So we can proceed with masking the entire picture except for the place where the number plate

mask = np.zeros(gray.shape, np.uint8)
new_image = cv2.drawContours(mask, [location], 0,255, -1)
new_image = cv2.bitwise_and(img, img, mask=mask)
The masked new image will appear something like below


2. Character Segmentation
The next step in Number Plate Recognition is to segment the license plate out of the image by cropping it and saving it as a new image. We can then use this image to detect the character in it. The code to crop the roi (Region of interest) image form the main image is shown below

(x,y) = np.where(mask==255)
(x1, y1) = (np.min(x), np.min(y))
(x2, y2) = (np.max(x), np.max(y))
cropped_image = gray[x1:x2+1, y1:y2+1]


The resulting image is shown below. Normally added to cropping the image, we can also gray it and edge it if required. This is done to improve the character recognition in next step. However I found that it works fine even with the original image.


They Use Easy OCR To Read Text


reader = easyocr.Reader(['en'])
result = reader.readtext(cropped_image)
result


they uses a Optical character recognition technique for reading and printing the text of the number plate in text format, that will be very usefull to understand

3. Character Recognition
The Final step in this Number Plate Recognition is to actually read the number plate information from the segmented image. We will use the pytesseract package to read characters from image, just like we did in previous tutorial. The code for the same is given below
