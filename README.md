# sqlalchemy-challenge

# Instructions
Congratulations! You've decided to treat yourself to a long holiday vacation in Honolulu, Hawaii. To help with your trip planning, you decide to do a climate analysis about the area. The following sections outline the steps that you need to take to accomplish this task.

# Part 1: Analyze and Explore the Climate Data
    In this section, used Python and SQLAlchemy to do a basic climate analysis and data exploration of the climate database.
    Used the SQLAlchemy create_engine() function to connect to the SQLite database.
    Used the SQLAlchemy automap_base() function to reflect the tables into classes, and then saved references to the classes named station and measurement.
    Linked Python to the database by creating a SQLAlchemy session.
    Performed a precipitation analysis and then a station analysis by completing the steps in the following two subsections.
        Precipitation Analysis:
        Found the most recent date in the dataset using that date, get the previous 12 months of precipitation data by querying the previous 12 months of data.
        Loaded the query results into a Pandas DataFrame. Explicitly set the column names.
        Sorted the DataFrame values by "date".
        Plotted the results by using the DataFrame plot method, as the following image shows:
        Used Pandas to print the summary statistics for the precipitation data.

        Station Analysis:
        Designed a query to calculate the total number of stations in the dataset.
        Designed a query to find the most-active stations by completing the following steps:
            Listed the stations and observation counts in descending order.
            Designed a query that calculates the lowest, highest, and average temperatures that filters on the most-active station id found in the previous query.
        Plotted the results as a histogram with bins=12.

    Closed the session

# Part 2: Design a Climate App
    #1 -created a homepage with a list of available routes:
        @app.route("/")
        def welcome():
            return (
                f"Welcome to the Honolulu Climate Analysis API!<br/>"
                f"Available Routes:<br/>"
                f"/api/v1.0/precipitation<br/>"
                f"/api/v1.0/stations<br/>"
                f"/api/v1.0/tobs<br/>"
                f"/api/v1.0/start (enter as YYYY-MM-DD)<br/>"
                f"/api/v1.0/start/end (enter as YYYY-MM-DD/YYYY-MM-DD)"

            )


    #2. -Convert the query results from your precipitation analysis to a dictionary using date as the key and prcp as the value.
        -Return the JSON representation of your dictionary.
            @app.route("/api/v1.0/precipitation")
            def precipitation():
                session = Session(engine)
                one_year= dt.date(2017, 8, 23)-dt.timedelta(days=365)
                prev_last_date = dt.date(one_year.year, one_year.month, one_year.day)
                results= session.query(Measurement.date, Measurement.prcp).filter(Measurement.date >= prev_last_date).order_by(Measurement.date.desc()).all()
                p_dict = dict(results)

                print(f"Results for Precipitation - {p_dict}")
                return jsonify(p_dict) 


    #3. -Return a JSON list of stations from the dataset.
            @app.route("/api/v1.0/stations")
            def stations():
                session = Session(engine)
                sel = [Station.station, Station.name, Station.latitude, Station.longitude, Station.elevation]
                queryresult = session.query(*sel).all()
                session.close()

            stations = []
            for station,name,lat,lon,el in queryresult:
                station_dict = {}
                station_dict["Station"] = station
                station_dict["Name"] = name
                station_dict["Lat"] = lat
                station_dict["Lon"] = lon
                station_dict["Elevation"] = el
                stations.append(station_dict)
            return jsonify(stations)


    #4. -Query the dates and temperature observations of the most-active station for the previous year of data.
        -Return a JSON list of temperature observations for the previous year.
            @app.route("/api/v1.0/tobs")
            def tobs():
                session = Session(engine)
                queryresult = session.query( Measurement.date, Measurement.tobs).filter(Measurement.station=='USC00519281')\
                .filter(Measurement.date>='2016-08-23').all()
                tob_obs = []
                for date, tobs in queryresult:
                    tobs_dict = {}
                    tobs_dict["Date"] = date
                    tobs_dict["Tobs"] = tobs
                    tob_obs.append(tobs_dict)
                return jsonify(tob_obs)


    #5. -Return a JSON list of the minimum temperature, the average temperature, and the maximum temperature for a specified start or start-end range.
        -For a specified start, calculate TMIN, TAVG, and TMAX for all the dates greater than or equal to the start date.
        -For a specified start date and end date, calculate TMIN, TAVG, and TMAX for the dates from the start date to the end date, inclusive.
            @app.route("/api/v1.0/<start>")
            def get_temps_start(start):
                session = Session(engine)
                results = session.query(func.min(Measurement.tobs), func.avg(Measurement.tobs), func.max(Measurement.tobs)).\
                        filter(Measurement.date >= start).all()
                session.close()

                temps = []
                for min_temp, avg_temp, max_temp in results:
                    temps_dict = {}
                    temps_dict['Min Temp'] = min_temp
                    temps_dict['Avg Temp'] = avg_temp
                    temps_dict['Max Temp'] = max_temp
                    temps.append(temps_dict)
                return jsonify(temps)


            @app.route("/api/v1.0/<start>/<end>")
            def get_temps_start_end(start, end):
                session = Session(engine)
                results = session.query(func.min(Measurement.tobs), func.avg(Measurement.tobs), func.max(Measurement.tobs)).\
                        filter(Measurement.date >= start).filter(Measurement.date <= end).all()
                session.close()

                temps = []
                for min_temp, avg_temp, max_temp in results:
                    temps_dict = {}
                    temps_dict['Min Temp'] = min_temp
                    temps_dict['Avg Temp'] = avg_temp
                    temps_dict['Max Temp'] = max_temp
                    temps.append(temps_dict)
                return jsonify(temps)


# File Submission:
    This assignment includes the following attchments:
      A Resources folder containing sqlite and csv files
      A Surfs Up folder containing a climate_starter.ipynb file and an app.py file.
      A License
      A readme file  


# Acknowledgements
    **Internet Search: I utilized Google Search and slack overflow to research coding concepts, algorithms, and best practices.
    **Gemini AI: I leveraged Gemini AI to generate code suggestions, debug errors, and provide explanations for complex code segments.
    **Support Staff: I recieved help from my instructor and class TA during office hours for help with debugging errors and further explanation for complex code segments.
    **Please note: While these tools were invaluable in my development process, the final code is the result of my analysis, testing, and refinement.