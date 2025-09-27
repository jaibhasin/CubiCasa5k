# Analysis of `get_wall_lines` Function Parameters

## Function Overview
`get_wall_lines` extracts wall line segments from wall heatmaps and room segmentation data by finding local maxima points and connecting them based on orientation constraints.

## Parameter Analysis

### 1. `wall_heatmaps`

**1. Runtime shape/dimensions:** 
- Shape: (N, H, W) where N=number of wall heatmap channels (typically 13), H=height, W=width
- Dtype: numpy.ndarray with float values

**2. Semantic meaning:** 
Heatmaps containing probability/activation values for wall junction points across different types and orientations.

**3. Values/range:** 
Float values typically in range [0, 1] after sigmoid activation, representing confidence of wall points at each pixel location.

**4. Parameter origin:** 
- File: `floortrans/post_prosessing.py` line 364: `wall_heatmaps = heatmaps[:13]`  
- File: `floortrans/models/hg_furukawa_original.py` line 207: `out[:, :21] = self.sigmoid(out[:, :21])`

**5. Usage inside function:**
- Line 226: `for i in range(len(wall_heatmaps)):`
- Line 228: `p = extract_local_max(wall_heatmaps[i], max_num_points, info, threshold, close_point_suppression=True)`
- Used for iterating through each heatmap channel to extract local maxima points

**6. Explicit assumptions:** 
- Must be indexable with integer indices
- Each channel must be 2D array compatible with `extract_local_max` function

### 2. `room_segmentation`

**1. Runtime shape/dimensions:**
- Shape: (C, H, W) where C=number of room classes, H=height, W=width  
- Dtype: numpy.ndarray with float values

**2. Semantic meaning:**
Multi-channel segmentation map containing probability distributions over room types for each pixel.

**3. Values/range:**
Probability values typically in range [0, 1], often output of softmax activation.

**4. Parameter origin:**
- File: `floortrans/post_prosessing.py` line 351: `heatmaps, room_seg, icon_seg = predictions`
- File: `floortrans/post_prosessing.py` line 1049: `rooms = F.softmax(rooms, 0)`

**5. Usage inside function:**
- Line 222: `_, height, width = room_segmentation.shape` - extract dimensions
- Line 244: `rooms_on_line = np.array([room_segmentation[:, i[0], i[1]] for i in line_pxls])` - sample along line pixels

**6. Explicit assumptions:**
- Must have exactly 3 dimensions with shape indexable as (C, H, W)
- Must support numpy-style indexing with coordinate pairs

### 3. `threshold`

**1. Runtime shape/dimensions:**
Scalar float value

**2. Semantic meaning:**
Minimum activation threshold for considering a point as a valid wall junction point.

**3. Values/range:**
Float value, typically in range [0, 1], commonly 0.2 based on usage example.

**4. Parameter origin:**
- File: `samples.ipynb`: `get_polygons((heatmaps, rooms, icons), 0.2, [1, 2])` - passed as 0.2

**5. Usage inside function:**
- Line 228: passed to `extract_local_max(wall_heatmaps[i], max_num_points, info, threshold, close_point_suppression=True)`
- Used to filter out low-confidence detections

**6. Explicit assumptions:**
- Must be comparable with heatmap values for thresholding

### 4. `wall_classes`

**1. Runtime shape/dimensions:**
List or array-like of integer class indices

**2. Semantic meaning:**
List of room/segmentation class IDs that represent wall areas.

**3. Values/range:**
Integer class indices, typically [2, 8] based on code analysis (wall=2, railing=8).

**4. Parameter origin:**
- File: `floortrans/post_prosessing.py` line 366: `wall_layers = [2, 8]`
- File: `floortrans/post_prosessing.py` line 367: passed to `get_wall_polygon`

**5. Usage inside function:**
- Line 246: `if segment in wall_classes:` - check if line segment passes through wall areas

**6. Explicit assumptions:**
- Must support `in` operator for membership testing
- Contains integer indices that correspond to segmentation classes

### 5. `point_orientations`

**1. Runtime shape/dimensions:**
Nested list structure: List[List[Tuple[int, ...]]] with shape (4, 4, variable)

**2. Semantic meaning:**
Defines allowed orientations for different types of junction points (4 point types, 4 orientations each).

**3. Values/range:**
Example structure: `[[(2,), (3,), (0,), (1,)], [(0,3), (0,1), (1,2), (2,3)], [(1,2,3), (0,2,3), (0,1,3), (0,1,2)], [(0,1,2,3)]]`

**4. Parameter origin:**
- File: `floortrans/post_prosessing.py` lines 355-358: defined as constant in `get_polygons`

**5. Usage inside function:**
- Line 231: passed to `calc_point_info(wall_points, gap, point_orientations, orientation_ranges, height, width)`
- Used in calc_point_info to determine valid connections between points

**6. Explicit assumptions:**
- Must be indexable as `point_orientations[point_type][orientation]`
- Inner tuples contain orientation indices 0-3

### 6. `orientation_ranges`

**1. Runtime shape/dimensions:**
List of lists with shape (4, 4) containing boundary coordinates

**2. Semantic meaning:**
Defines spatial search ranges for each orientation direction (north, east, south, west).

**3. Values/range:**
Example: `[[width, 0, 0, 0], [width, height, width, 0], [width, height, 0, height], [0, height, 0, 0]]`

**4. Parameter origin:**
- File: `floortrans/post_prosessing.py` lines 359-362: defined as constant in `get_polygons`

**5. Usage inside function:**
- Line 231: passed to `calc_point_info(wall_points, gap, point_orientations, orientation_ranges, height, width)`
- Used to constrain point connection search areas

**6. Explicit assumptions:**
- Must be indexable as `orientation_ranges[orientation]`
- Contains numeric boundary values for spatial constraints

### 7. `max_num_points` (default=100)

**1. Runtime shape/dimensions:**
Scalar integer value

**2. Semantic meaning:**
Maximum number of local maxima points to extract from each wall heatmap channel.

**3. Values/range:**
Positive integer, default value is 100.

**4. Parameter origin:**
- Function default parameter, no explicit override found in traced code paths

**5. Usage inside function:**
- Line 228: passed to `extract_local_max(wall_heatmaps[i], max_num_points, info, threshold, close_point_suppression=True)`

**6. Explicit assumptions:**
- Must be positive integer for loop termination in extract_local_max

## Function Summary
**One-line summary:** Extracts connected wall line segments by finding local maxima in wall heatmaps and connecting them based on orientation constraints and room segmentation validation.

## Minimal Example
**Input shapes:**
- wall_heatmaps: (13, 256, 256) - 13 wall heatmap channels
- room_segmentation: (12, 256, 256) - 12 room classes  
- threshold: 0.2
- wall_classes: [2, 8]
- point_orientations: as defined in get_polygons
- orientation_ranges: as defined in get_polygons
- max_num_points: 100

**Expected output shape:** 
- wall_lines: List of tuples (point1_idx, point2_idx, segment_class)
- wall_points: List of point coordinates [x, y, type, orientation, confidence]  
- wall_point_orientation_lines_map: List of dicts mapping orientations to line indices

## Files to Read for Shape Confirmation
- `floortrans/post_prosessing.py` - Function implementation and get_polygons usage
- `floortrans/models/hg_furukawa_original.py` - Model output shapes  
- `samples.ipynb` - Usage examples with concrete parameters
- `floortrans/loaders/house.py` - Data loading and heatmap generation