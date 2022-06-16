# Project 2 -Ames Housing Data

In this project notebook, we look at the housing market in Ames, IA, in particular, sales from 2006 to 2010. The houses being sold have 81 features about them. These include how large the property is, the types of rooms, where the property is, fencing, the slope of the land, and various other factors. I wanted to devise a model that was good at predicting the sales price from these characteristics as a first goal. Beyond that, I wanted to look at 2 elements in particular and see how good they are at predicting the sales price relative to other predictive models and see if there was any merit to how they were used- namely the zoning of a property and the specific neighborhood it was in.

## Data Sources

This data comes from a study of data in Ames using county assessor data. The entire data dictionary can be found [here](http://jse.amstat.org/v19n3/decock/DataDocumentation.txt) Beyond this, I made some modifications to the data. I mostly tried to trim columns that didn't add information for whatever reason. But beyond that, I also encoded all nominal data using a set of one hot encoders that meant that any nominal data was converted into a series of columns for each unique data entry with only a 1 for the data entry they matched with. If they didn't match any of the entries, they would have a 0 for all rows. I also took the log of the variable for living area in  square feet, the lot area in sq feet, and the target variable- sale price. This is because all of those variables had skews that made them non-normal. Furthermore, a log-transformation is often used for prices and areas for economic reasons related to marginal costs. The final trasnformation was to find the interaction terms between all numerical columns. I also created a 'months since 1990' to better capture time data of sales. Categorical encoded data was avoided to avoid feature bloat and in most cases, does not provide new information. There were also no square terms- it was felt that these terms seemed not to have any theoretical justification for why they would help the model and thus would lead to feature bloat. There are 2 main datasets- 1 used in the general model for salesprice, and the more focused set looking into zoning and neighborhoods. I'll avoid listing every ordinal and polynomial term, but simply state if it was used as an ordinal or polynomial, and what sets the data was in below:
Note that type int/float implies polynomial, while str implies ordinal 

|Feature|Type|Gen_Dataset|Spec_Dataset|Description|
|---|---|---|---|---|
|**ms_zoning**|*str*|Yes|Yes|Zoning Code on property lot|
|**lot_area**|*int*|Yes|Yes|Area of lot in sq. feet (log-transformed)|
|**utilities**|*str*|Yes|No|Types of Utilities available|
|**lot_config**|*str*|Yes|No|Lot coniguration (cul-de-sac,inside,corner)|
|**neighborhood**|*str*|Yes|Yes|Neighborhood in Ames where lot is|
|**bldg_type**|*str*|Yes|No|Type of building-SFH,duplex,townhouse,etc|
|**year_built**|*int*|Yes|No|Year property was built|
|**total_bsmt_sf**|*int*|Yes|No|Total area of basement in sq. feet|
|**heating**|*str*|Yes|No|Type of heating|
|**central_air**|*str*|Yes|No|Whether the property has central AC|
|**1st_flr_sf**|*int*|Yes|No|Square footage of 1st floor of property|
|**2nd_floor_sf**|*int*|Yes|No|Square footage of 2nd floor of property|
|**gr_liv_area**|*int*|Yes|Yes|Area of dwelling sq. feet (log-transformed)|
|**bsmt_full_bath**|*int*|Yes|No| Number of bathrooms in the basement|
|**full_bath**|*int*|Yes|No| Number of bathrooms above-ground|
|**half-bath**|*int*|Yes|No| Number of half-bathrooms above-ground|
|**bedroom_abvgr**|*int*|Yes|No| Number of bedrooms above-ground|
|**kitchen_abvgr**|*int*|Yes|Yes|Number of kitchens above-ground|
|**totrms_abvgrd**|*int*|Yes|No|Number of rooms above-ground|
|**fireplaces**|*int*|Yes|No| Number of fireplaces|
|**garage_type**|*str*|Yes|No|Type of garage on property|
|**garage_cars**|*int*|Yes|Yes|How many cars can fit in garage|
|**garage_area**|*int*|Yes|No|Garage area in square feet|
|**paved_drive**|*str*|Yes|No|Whether or not the driveway is paved and with what|
|**screen_porch**|*int*|Yes|No|Screen porch area in sq feet|
|**pool_area**|*int*|Yes|No| Pool area in square feet|
|**pool_qc**|*str*|Yes|No|Subjective quality of pool|
|**fence**|*str*|Yes|No| Checking if there is a fence and fence quality|
|**misc_feature**|*str*|Yes|No| Random other potential features in a house|, such as an elevator|
|**misc_val**|*int*|Yes|No|Value of said feature to house|
|**sale_type**|*str*|Yes|No| Type of sale  of house|
|**mo_since_1990**|*int*|Yes|No|Number of months from Jan 1990 to sale date|

No other data sources were used

## Procedure

The above covers the ordinal/polynomial feature encoding and feature engineering used. Beyond that, a TransformedTargetRegressor was used to convert y to log(y) and normalized the data. Then, I used 3 different methods of cutting down on regressor variables so as to lower variance- RecursiveFeatureElimination, Lasso, and Ridge Fitting. For RecursiveFeatureElimination, I put it through a gridsearch to check over hyperparameters- in particular, I was curious on how many features should be left after elimination, whether I should add a column of 1s, and whether I should keep the features to just interaction terms. It seems that keeping 30% of the features and keeping the column of 1s and keeping the polynomial features to interaction_only were the right moves. Then, I tested on ridge and lasso models on alpha to see where the maximum would be there. After assessing these models, I tightened my model to just a couple of the most predictive variables and lot_zoning/neighborhoods. Then, I tried to find which neighborhoods had the most predictive value in determining sale_price and how that compared to other predictive variables

## Results

We found that RFE was the best regressor to remove excess variables and maintain a small residual over the test set. The most important variables tended to be polynomial functions involving the living area: 'gr_lv_area' and various rooms, such as the number of bathrooms, or the size of the garage, suggesting that people really value their house's size and pay significantly more for it. Beyond that, regular 'gr_lv_area' was also highly predictive. We found that for zoning- living in a low-density Residential area is most closely related with high prices. That being said, it seemed to also be well correlated with house-size, so it's hard to know if that's just a proxy for that- it can be the case that large houses in high density neighborhoods can be extremely expensive. It's hard to properly evaluate the coefficient given that we took a scalar of the function and passed a log function on our target variable- but low density tended to have the highest linear coefficients when fitted. Beyond that, neighborhoods tended to have even less predictive power for prices. The normalized coefficients were low, and the range between normalized coefficients being a fraction of 'gr_lv_area' suggests that even the maximum prediction between the nicest neighborhood and the worst neighborhood pales in comparison to simply the area of the house. If there were recommendations, they would be that neighborhoods don't provide amenities disproportionate to their location in Ames, Iowa- that is, it's fine to be in an 'uncool' part of Ames, as there isn't too big of a difference. Beyond that, the recommendation would be to focus on the basics on housing- bigger spaces, more bathrooms and bedrooms, etc are all more important to the saleprice than any of the myriad other minor factors that can take up a lot of your time or planning.

## Future Endeavours

If I sought to look at this data set more, I would look into a more careful examination of what features to use, and not as randomly pick them. It would be better to have total R^2 or RMSE scores of which features seem to be initially quite useful. Furthermore, a correlation matrix among X could help weed out multicollinearity. Given this information, I wonder if I would get different results on how important neighborhoods are. Beyond that, I might want to use an elastic net to get a different view on the feature elimination process, and use that to see which features pop out as important. If I had a lot of time, I would use more compute to really explode the polynomial features column and see if any relationships popped out, and then see how powerful feature elimination would be, and what new,unknown relationships would be there in the data

