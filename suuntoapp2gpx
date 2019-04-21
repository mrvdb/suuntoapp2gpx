#!/usr/bin/env python
# -*- mode:python -*-
# Convert suunto zip files (which contain json files) to a gpx file
# Until Suunto gets their act together and let me have my own data normally.
# Needs: python 3.7

import sys
import gpxpy
import gpxpy.gpx
import json
import datetime
import zipfile
import math

# The relevant json snippets which are on a json array 'Samples'
# looks like this:

#  {
#    "Attributes": {
#        "suunto/sml": {
#            "Sample": {
#                "GPSAltitude": 94,                     <-- assuming meters
#                "Latitude": 0.88924328690504395,       <-- this value is weird
#                "Longitude": 0.10261437316962527,      <-- this value is weird
#                "UTC": "2019-04-18T07:25:29.000+00:00" <-- assumed what gps thinks of time?
#            }
#        }
#    },
#    "Source": "suunto-123456789",                      <-- id of watch
#    "TimeISO8601": "2019-04-18T09:25:29.000+02:00"     <-- assumed what watch thinks of time?
#  },

# Suunto specific stuff
SUUNTO_SAMPLES = 'samples.json'


def gpx_track(zip):
    # Three keys make a trackpoint
    TRKPT = ('Latitude', 'Longitude', 'UTC')

    # Create the main gpx object
    gpx = gpxpy.gpx.GPX()

    # We are getting one track presumably, from the points we recorded
    track = gpxpy.gpx.GPXTrack()
    gpx.tracks.append(track)

    # Points are added to segments (take breaks into account later?)
    segment = gpxpy.gpx.GPXTrackSegment()
    track.segments.append(segment)

    # Read in the json file
    with zip.open(SUUNTO_SAMPLES, 'r') as json_data:
        suunto_json = json.load(json_data)

        for s in suunto_json["Samples"]:
            sample = s["Attributes"]["suunto/sml"]["Sample"]
            # See if we have enough for a trackpoint
            if all(key in sample for key in TRKPT):
                # Lat and long are floats between 0 and 1
                # To decimal degrees = * 180 / PI
                lat =  (sample["Latitude"] * 180) / math.pi # Used to be * 57.2958320392
                lon = (sample["Longitude"] * 180) / math.pi # Used to be * 57.2952851192

                # I'm assuming this is in meters, and we can use it as is
                ele = sample['GPSAltitude'] if 'GPSAltitude' in sample else 0

                time = datetime.datetime.fromisoformat(sample['UTC'])

                # Create the gpx point
                point = gpxpy.gpx.GPXTrackPoint(latitude=lat, longitude=lon,
                                                time=time, elevation=ele)
                segment.points.append(point)

    # Write out the gpx file
    with open(zip.filename.replace(".zip", ".gpx"), "w") as out:
        out.write(gpx.to_xml())


if __name__ == "__main__":
    # Argument is expected to be a zip file
    with zipfile.ZipFile(sys.argv[1]) as zip:
        # There should be a samples.json in the zip file
        if SUUNTO_SAMPLES in zip.namelist():
            gpx_track(zip)
