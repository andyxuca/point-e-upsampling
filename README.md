## Introduction

Our objective for the extension was to enhance the Point-E library by implementing recursive upsampling of point clouds. The motivation behind this was to generate higher-resolution point clouds with increased detail and density, improving the visual quality of 3D models produced by the library. The original upsampler implementation is limited to the generation of 3072 more points. We attempt to increase the number of generated points by recursively subsampling results and upsampling, and merging all points to create the final point clouds. 

## Modifications Made

We created a recursive_upsample function to iteratively upsample the point cloud. Each iteration involves downsampling the current point cloud and then using the upsampler model to generate new points, which are merged back into the original point cloud. We hope that by doing this, we can get higher resolution point clouds without loss of feature quality.

We chose to first downsample the original point cloud to 1024 points while preserving all channels to ensure consistency. We used the built-in “farthest_point_sample” function to sample uniformly across space. It works by first choosing a random starting point. Each successive point is the farthest point in the point cloud from the currently chosen set of points. We hope that this approach will prevent us from having missing features or large holes while downsampling.

Then, we manipulated the 1024 points into tensors with their color channels so that they could be inputted into the upsampler, which generated 3072 new points. These points were merged into the original point cloud. Additionally, we nearest neighbors to assign colors for points in the point clouds to avoid uncolored holes in the generated point cloud. We did this after each upsampling iteration, combining the newly generated points with the existing point cloud. Our hope was that this ensembling of outputs would help us create successively denser, and more accurate point clouds. 

Our code can be accessed here: https://colab.research.google.com/drive/11otxIgmyJMtsKE0CyG_cFT8qByEtDhac?usp=sharing


## Original Results:

Original results had 4096 points maximum, causing limited point density. This was a sacrifice the authors were willing to make in exchange for extremely high computational efficiency. Generated point clouds lacked certain details for high-resolution applications, and visual artifacts and less smooth surfaces were noticeable in the models. This can be somewhat mitigated by converting the point clouds into rough meshes before processing them, but we wished to increase the quality of the point clouds themselves. 

Figure 1. Original results of 4096 points


## Qualitative Results:

Figure 2. Final results containing 1024 + 3072*3 = 10240 points

Figure 3: Meshes generated from original point cloud (left) and upsampled point cloud (right)

As we saw with the cubestack and corgi examples, we managed to achieve higher point densities through recursive upsampling. We managed to do so while retaining many important features, as well as smoothing out some of the holes and color gaps. Thus, we are able to enhance the visual detail, as well as add more density. To help confirm that the fidelity increases, we converted the point clouds into meshes for qualitative comparisons. We see that many of the holes in the ears and paws are patched up, and the general shape is more similar to an actual corgi, though the tail of the mesh still possesses artifacts in the tails for both meshes. 

Because we are running the upsampler model multiple times, the generation time increases by a factor of two to three. However, as this model is already so efficient, the trade-off between a slight increase in generation time and a significant enhancement in point cloud quality could be worthwhile.

## Quantitative Results:
To get some more solid results, we analyzed our point clouds and meshes using various metrics. One metric we tried was the P-FID, or Point-Frechét Inception Distance, as described in the original paper. However, our results showed that the original point cloud had a P-FID of 36.95 and our upsampled point cloud had a P-FID of 43.64. However, since upsampling could significantly change the distribution since sparse volume is being filled, the P-FID score might not accurately represent how well our point cloud represents various features. We were also unable to find the authors’ ground truth, so we used the base output from the 1B parameter model as the ground truth. The distance may simply represent that our distribution of denser points is not as similar as the base output, which is to be expected.

While analyzing the meshes, we measured five main metrics: vertex count, face count, smoothness score, surface area variation, and surface normal consistency. Smoothness score was the average of angles between adjacent faces in the mesh, and surface normal consistency averages the dot product between adjacent surface normals. In both metrics, our mesh outperformed the original, indicating that our surface was smoother and contained less artifacts. The surface area variation measures the uniformity of face size, and once again our upsampled mesh outperformed the original. Because this was achievable with a lower vertex and face count, our model can represent more complex features with smoother surfaces and more consistency, a good preliminary result.

Figure 4: Mesh metric comparison

## Conclusion

As we continue to modify our code and experiment, one possibility we can explore is to successively decrease the radius of the points as we go, such that we aren’t left with largely overlapping spheres. It is also possible that more sophisticated post-processing of the point clouds can help during the conditioning. For example, it might be possible to upsample small regions at a time. One approach could be to take cubic samples of points and upsampling them by three times the number of points. Another possibility would be to develop some measure of “sparseness” in the point cloud. We would then take the sparse areas and upsample them one at a time. In this way, we might be able to add detail at a more granular level.

