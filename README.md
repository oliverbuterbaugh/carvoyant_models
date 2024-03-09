# carvoyant_models

## Intro

The Carvoyant app uses six tree ensemble regressor (random forest) models, one for each carpark, trained on roughly five weeks of data. Each model has six features: Precipitation, WindSpeed, DayOfWeek, MinutesPastMidnight, WeedendWeekday, and TimeOfDay. The latter two are categorical features representing progress through the workweek and workday, respectively; these are calculated by the app using the day of week and time inputs.

## Data collection

Government of Jersey publishes the live data of Green St, Minden Pl, Patriotic St, Sand St, Pier Rd, and Les Jardin carparks to https://www.gov.je/Travel/Motoring/Parking/pages/carparkspaces.aspx, which, on brief inspection of the source, gets its data from the public API https://sojpublicdata.blob.core.windows.net/sojpublicdata/carpark-data.json. I've included the clean data (with precipitation and wind speed added) up to date with the models.

### clean_carpark_data.csv

- Available_Spaces represents the spaces left in the carpark
- DayOfWeek represents the day of week, with 0 being monday and 6 being sunday
- MinutesPastMidnight represents the time in minutes
- Precipitation is calculated hourly in mm
- WindSpeed is calculated hourly in km/h
- CarparkCodeInt is the carpark represented as 0 - 5, in the order Green St, Minden Pl, Patriotic St, Sand St, Pier Rd, Les Jardin
- WeekendWeekday is represented as a 1 for weekday and 0 for weekend
- TimeOfDay is represented as a 0 for before 06:00, a 1 for between 06:00 and 18:00, and a 2 for after 18:00

## Usage in Xcode project / app playground

Drag each .mlmodel file into the sidebar, ensuring they're added to the target. On build, a class for each model will be automatically generated. You can then make a simple call to the model using the built-in input class.

```
import CoreML

let model = try greenst(configuration: MLModelConfiguration())

let input = greenstInput(
    Precipitation: 0.0, // 0 mm rainfall
    WindSpeed: 20.0, // 20 km/h windspeed
    DayOfWeek: 0, // monday
    MinutesPastMidnight: 600, // 10:00 am
    CarparkCodeInt: 5, // Les Jardin
    WeekdayWeekend: 1, // weekday
    TimeOfDay: 1 // within working hours
)

let result = try model.prediction(input: input)
print(Int(result.Available_Spaces))
```

For an app playground, which doesn't support .mlmodel resources and therefore won't generate the model class, just use a dummy Xcode project to build and generate the class, then copy it into your app playground. Change the urlOfModelInThisBundle class to this in order to get it working on the iPad Playground app:
```
class var urlOfModelInThisBundle : URL {
  let resPath = Bundle(for: self).url(forResource: "greenst", withExtension: "mlmodel")!
  return try! MLModel.compileModel(at: resPath)
}
```
