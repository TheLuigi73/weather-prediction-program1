# weather-prediction-program1
//This program will predict the weather for a given day by analyzing the weather on that day for the past 20 years in (Houston) .

#include <iostream>
#include <fstream>
#include <sstream>
#include <string>
#include <vector>
#include <iomanip>
#include <cmath>

using namespace std;

int main() {
    ifstream inputFile("4284751.csv");
    if (!inputFile.is_open()) {
        cerr << "Error: Could not open 4284751.csv. Ensure it is in the same folder." << endl;
        return 1;
    }

    string targetMonthDay;
    cout << "Please Enter the Month and Day you are interested in (example: 5-23): ";
    cin >> targetMonthDay;

    // Separate input into target month and day
    size_t dashPos = targetMonthDay.find('-');
    int targetMonth = stoi(targetMonthDay.substr(0, dashPos));
    int targetDay = stoi(targetMonthDay.substr(dashPos + 1));

    string line, header;
    getline(inputFile, header); // Skip header row

    // Accumulators for calculations
    double sumMaxT = 0, sumMinT = 0, sumWind = 0, sumRainAmt = 0;
    int countDays = 0, rainDays = 0, thunderDays = 0, rainAmtCount = 0;

    while (getline(inputFile, line)) {
        stringstream ss(line);
        string cell;
        vector<string> row;

        while (getline(ss, cell, ',')) {
            // Remove quotes if present
            if (!cell.empty() && cell.front() == '"') cell.erase(0, 1);
            if (!cell.empty() && cell.back() == '"') cell.pop_back();
            row.push_back(cell);
        }

        // Standard NOAA CSV column mapping (Adjust index if your file differs)
        // 0:STATION, 1:DATE, 2:AWND, 3:PRCP, 4:TMAX, 5:TMIN, 6:WT03
        if (row.size() < 6) continue;

        string dateStr = row[1]; // Expected format YYYYMMDD or YYYY-MM-DD
        int rowYear, rowMonth, rowDay;
       
        if (dateStr.find('-') != string::npos) {
            sscanf(dateStr.c_str(), "%d-%d-%d", &rowYear, &rowMonth, &rowDay);
        } else {
            int dateVal = stoi(dateStr);
            rowMonth = (dateVal % 10000) / 100;
            rowDay = dateVal % 100;
        }

        if (rowMonth == targetMonth && rowDay == targetDay) {
            double wind = row[2].empty() ? 0 : stod(row[2]);
            double prcp = row[3].empty() ? 0 : stod(row[3]);
            double tmax = row[4].empty() ? 0 : stod(row[4]);
            double tmin = row[5].empty() ? 0 : stod(row[5]);
            bool thunder = (row.size() > 6 && row[6] == "1");

            sumMaxT += tmax;
            sumMinT += tmin;
            sumWind += wind;
            if (prcp > 0) {
                rainDays++;
                sumRainAmt += prcp;
                rainAmtCount++;
            }
            if (thunder) thunderDays++;
            countDays++;
        }
    }

    if (countDays == 0) {
        cout << "No data found for " << targetMonthDay << "." << endl;
        return 0;
    }

    // Calculations
    int avgMaxF = round(sumMaxT / countDays);
    int avgMaxC = round((avgMaxF - 32) * 5.0 / 9.0);
    int avgMinF = round(sumMinT / countDays);
    int avgMinC = round((avgMinF - 32) * 5.0 / 9.0);
    int avgWind = round(sumWind / countDays);
    int probRain = round((static_cast<double>(rainDays) / countDays) * 100);
    int probThunder = round((static_cast<double>(thunderDays) / countDays) * 100);
    double avgPrcp = (rainAmtCount > 0) ? (sumRainAmt / rainAmtCount) : 0.0;

    // Output
    cout << "\nPredicted Weather for " << targetMonthDay << ":" << endl;
    cout << "- Maximum Temperature: " << avgMaxF << " F/ " << avgMaxC << " C" << endl;
    cout << "- Minimum Temperature: " << avgMinF << " F/ " << avgMinC << " C" << endl;
    cout << "- Average Wind Speed: " << avgWind << " MPH" << endl;
    cout << "- Probability of Rain: " << probRain << "%" << endl;
    cout << "- Probability of Thunder: " << probThunder << "%" << endl;
    cout << fixed << setprecision(2) << "- Amount of Precipitation if it Rains: ." << (int)(avgPrcp*100) << " in" << endl;

    inputFile.close();
    return 0;
}

