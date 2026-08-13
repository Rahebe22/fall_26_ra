The following is the description from our FTW V2 grants that details the work we are supposed to do: 

> ...we will identify and implement shape-based methods that can be used to both assess and improve the quality of FTW field boundary maps. We will do so by building and improving on approaches currently used in our Mapping Africa workflow and downstream map-based assessments (Xiong et al, forthcoming). 
> 
> Shape-based information and approaches for assessing and improving already generated maps will be further developed. Measures such as fractal dimension and compactness will be evaluated at larger scales to assess their reliability for identifying segmentation errors. Approaches for separating fields that are under-segmented (e.g. narrow-neck splitting) will be further developed.
> 
> The selected set of shape metrics will also be evaluated for their ability to assess relative performance improvements in field mapping models. These can be integrated as additional performance metrics in the model evaluation code, and also for assessing map quality over unlabelled regions. For example, a model that is more effective at separating adjacent fields should return a histogram of compactness values that are shifted further to the right than that of a less effective model. The shape metrics that indicate segmentation quality will likely vary between maps based on Sentinel-2 (FTW standard) and Planet (FTW Planet), thus we will also examine how shape metric-based map quality indicators vary by input dataset. Additionally, a lightweight model (e.g. Random Forests) based on shape metrics will be evaluated for efficacy
in removing false positive fields.
> 
> Shape metrics may also prove useful within the model training process, by integrating them into the model’s loss function. We have started exploring how to add a shape-based loss term to Tversky Focal Loss, and will continue to research this possibility under this project. This will include identifying existing approaches in the literature, such as from medical image segmentation, as well as formulating novel loss terms based on compactness and similar shape metrics.
> 
> Clark will be responsible for:
> 
> - Further research, testing, and selection of recommended shape metric suite for map quality assessment and reporting;
> - Developing and testing polygon simplification, regularization, and splitting routines that can be integrated into the FTW ecosystem;
> - Developing and testing shape metric-based models for flagging and removing false positives;
> - Developing, testing, and potentially integrating shape-based loss terms with existing area-based loss functions;
> - Demonstrating the utility of shape-based metrics in assessing model performance gains, post-inference map improvements, and for providing information on map quality and usability, through papers or other media.

We need to define the tasks within this. Here is a first list of more specific tasks within this, which we can refine going forward:

1. Audit and summary into report of shape metrics assessed across various projects, including:
    - instancemaker
    - as used in Xiong et al (2026) to identify under-segmentation
    - Ones tested by @Rahebe22 in #3 
    - Reported in Fields of the Planet (Panoptic Quality)
2. Evaluation of most informative/parsimonious set for different map-based applications. 
    - removing false positives from produced maps
    - flagging or otherwise quantifying under-segmentation
    - for detecting different types of agriculture (center pivots, etc)
    - As candidates for integration into loss functions (see @Gregory-Essuman's work on this with compactness)
    - For large-scale or chip-based evaluation, which set of measures are most complementary to object recall/precision/F1/panoptic quality. 
3. Polygon improvement approaches:
    - Other measures: what are the best simplication approaches (reducing jagged edges, making polygons more regular)--can we do better than current instancemaker, or approaches used in FTP/FTW: https://github.com/taylor-geospatial/fields-of-the-planet/tree/main
    - Splitting routines. Can we reliably split under-segmented polygons at narrow necks
    - Can we improve quality during training time (e.g. with loss)? 

Evaluations/tests will need to be done not just on Planet data (e.g. ours), but on FTW/FTP data also. We will need to make our model code base and procedure closer to theirs, and use theirs at times also. 

This will be built out further into more specific issues, but for now let's start with point 1. 


