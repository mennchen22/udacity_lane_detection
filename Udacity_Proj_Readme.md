# **Finding Lane Lines on the Road** 

[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

---

### Pipeline reflection

In the first instance the incoming line array will be checked squeezed for easier data manipulation by indicies

    # remove unwanted dim [[[]]] --> [[]]
    lines = np.squeeze(lines)

Afterwards the slope *((y2-y1)/(x2-x1))* is calculated as a combination of subtractions and divisions. The result will be appended as an additional column to the line array

    # calculate slope
    result = np.expand_dims(np.array((lines[:, 3] - lines[:, 1]) / (lines[:, 2] - lines[:, 0])), axis=1)
    lines = np.append(lines, result, axis=1)

The left line is seperated by the right one by filtering the array with the slope significant

    # get left and right
    side_filter = lines[:, -1] < 0
    left = lines[~side_filter]
    right = lines[side_filter]

Because the operation is similar for both side we pass the filtered arrays to a new method *get_line_for_side()*

    # get single line from avarage or median and plot it on the image
    get_line_for_side(img, left, mode, color, thickness)
    get_line_for_side(img, right, mode, color, thickness)

This function checks if data is within the array before calculating anything
    # bounding check
    if len(lines) == 0 or lines.shape[0] == 0 or lines.shape[1] != 5:
        print("[Draw Lines] No lines detected, skip")
        return

    # get image shape
    img_shape = img.shape
    slope = point_x2 = point_y2 = 0.
Given by the user choice the average or the median of the points and slopes, could be processed

    if mode == 0:
        point_x2 = np.median(lines[:, 2])
        point_y2 = np.median(lines[:, 3])
        slope = np.median(lines[:, -1])
    if mode == 1:
        point_x2 = np.average(lines[:, 2])
        point_y2 = np.average(lines[:, 3])
        slope = np.average(lines[:, -1])

Based on the formula y = mx + c the missing c is calculated. If the values is out of bounds or not a number we will skip this call

    # get c offset in y = mx + c --> c = y - (mx)
    c = point_y2 - (slope * point_x2)

    # catch out of bound values 
    if np.isnan(c) or np.isinf(c) or np.isnan(slope) or np.isinf(slope) :
        print(f"[Draw Lines] Slope or c is infinity [s={slope}, c ={c}]")
        print(lines)
        print("================================")
        return

Because we need a slice of the line within the image region we cut the line in the y axis scale from the bottom to a given ratio of the y axis.

    # calculate x by given y  x = (y - c)/m
    point_bot = np.array([(img_shape[0] - c) / slope, img_shape[0]], dtype=np.int32)
    point_top = np.array([((img_shape[0] * (1 - region_top_ratio)) - c) / slope, img_shape[0] * (1 - region_top_ratio)], dtype=np.int32)

At the end the line is drawn onto the image with a given color and thickness.

    cv2.line(img, (point_bot[0], point_bot[1]), (point_top[0], point_top[1]), color, thickness)


### 2. Potential shortcomings with the current pipeline


One potential shortcoming would be what would happen when no lines are detected. In this case nothing is given back to the system. 

Another shortcoming could be the change of lighting conditions in the image, so that the predefined values for the hogh transformation will not fit within the chnaged enviroment

This problem also exists with large road segment gaps and defect road painting with small fractions instead of a solid line


### 3. Suggest possible improvements to your pipeline

A possible improvement would be to modify the parameters based on the image brightness.

Another potential improvement could be to store detected lines from the last image and predict a new position within the new image if no line is detected.
