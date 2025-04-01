---
title: Welcome to my blog
---
import React, { useState, useEffect } from 'react';
import { LineChart, Line, XAxis, YAxis, CartesianGrid, Tooltip, Legend, ResponsiveContainer, ReferenceLine } from 'recharts';

const VehicleTCOCalculator = () => {
  // Default values for two vehicles
  const [vehicle1, setVehicle1] = useState({
    name: 'Vehicle 1 (Gas)',
    msrp: 36965,
    fuelType: 'gas',
    fuelEfficiency: 20, // mpg
    fuelCost: 4.22, // $ per gallon
    annualMaintenance: 800,
    majorMaintenanceCost: 1500,
    majorMaintenanceYears: [4, 8, 12],
    annualMiles: 12000,
    batteryReplacementCost: 0,
    batteryReplacementYear: 0,
    color: '#4173B4',
  });

  const [vehicle2, setVehicle2] = useState({
    name: 'Vehicle 2 (Electric)',
    msrp: 65090,
    fuelType: 'electric',
    fuelEfficiency: 2.45, // miles per kWh
    fuelCost: 0.1343, // $ per kWh
    annualMaintenance: 400,
    majorMaintenanceCost: 1000,
    majorMaintenanceYears: [4, 8, 12],
    annualMiles: 12000,
    batteryReplacementCost: 12000,
    batteryReplacementYear: 10,
    color: '#41A154',
  });

  const [years, setYears] = useState(13);
  const [taxCredit, setTaxCredit] = useState(0);
  const [showAdvanced, setShowAdvanced] = useState(false);
  const [data, setData] = useState([]);
  const [crossoverYear, setCrossoverYear] = useState(null);

  // Helper function to calculate annual fuel costs
  const calculateAnnualFuelCost = (vehicle) => {
    if (vehicle.fuelType === 'gas') {
      return (vehicle.annualMiles / vehicle.fuelEfficiency) * vehicle.fuelCost;
    } else if (vehicle.fuelType === 'electric') {
      return (vehicle.annualMiles / vehicle.fuelEfficiency) * vehicle.fuelCost;
    } else if (vehicle.fuelType === 'hybrid') {
      return (vehicle.annualMiles / vehicle.fuelEfficiency) * vehicle.fuelCost;
    } else if (vehicle.fuelType === 'phev') {
      // Simplified PHEV calculation assuming 60% electric, 40% gas
      const electricMiles = vehicle.annualMiles * 0.6;
      const gasMiles = vehicle.annualMiles * 0.4;
      const electricCost = (electricMiles / vehicle.electricEfficiency) * vehicle.electricityCost;
      const gasCost = (gasMiles / vehicle.gasEfficiency) * vehicle.gasCost;
      return electricCost + gasCost;
    }
    return 0;
  };

  // Calculate total cost of ownership data
  useEffect(() => {
    const calculatedData = [];
    let foundCrossover = false;
    let crossoverPoint = null;
    
    // Apply tax credit to the initial price if it's an EV or PHEV
    const v2InitialPriceAfterCredit = 
      (vehicle2.fuelType === 'electric' || vehicle2.fuelType === 'phev') 
        ? Math.max(vehicle2.msrp - taxCredit, 0) 
        : vehicle2.msrp;

    for (let year = 0; year <= years; year++) {
      // Calculate annual fuel costs
      const v1AnnualFuelCost = calculateAnnualFuelCost(vehicle1);
      const v2AnnualFuelCost = calculateAnnualFuelCost(vehicle2);

      // Check for major maintenance in this year
      const v1MajorMaintenance = vehicle1.majorMaintenanceYears.includes(year) ? vehicle1.majorMaintenanceCost : 0;
      const v2MajorMaintenance = vehicle2.majorMaintenanceYears.includes(year) ? vehicle2.majorMaintenanceCost : 0;

      // Check for battery replacement in this year
      const v1BatteryCost = year === vehicle1.batteryReplacementYear ? vehicle1.batteryReplacementCost : 0;
      const v2BatteryCost = year === vehicle2.batteryReplacementYear ? vehicle2.batteryReplacementCost : 0;

      // Calculate cumulative costs for current year
      const v1Cost = year === 0 
        ? vehicle1.msrp 
        : calculatedData[year-1][vehicle1.name] + v1AnnualFuelCost + vehicle1.annualMaintenance + v1MajorMaintenance + v1BatteryCost;
      
      const v2Cost = year === 0 
        ? v2InitialPriceAfterCredit
        : calculatedData[year-1][vehicle2.name] + v2AnnualFuelCost + vehicle2.annualMaintenance + v2MajorMaintenance + v2BatteryCost;
      
      // Check for crossover point
      if (!foundCrossover && year > 0 && calculatedData[year-1][vehicle1.name] < calculatedData[year-1][vehicle2.name] && v1Cost >= v2Cost) {
        foundCrossover = true;
        crossoverPoint = year;
      }
      
      const yearData = {
        year: year,
        [vehicle1.name]: v1Cost,
        [vehicle2.name]: v2Cost,
      };
      
      calculatedData.push(yearData);
    }
    
    setData(calculatedData);
    setCrossoverYear(crossoverPoint);
  }, [vehicle1, vehicle2, years, taxCredit]);

  // Handle input change for vehicle 1
  const handleVehicle1Change = (e) => {
    const { name, value } = e.target;
    setVehicle1({
      ...vehicle1,
      [name]: name === 'name' ? value : parseFloat(value),
    });
  };

  // Handle input change for vehicle 2
  const handleVehicle2Change = (e) => {
    const { name, value } = e.target;
    setVehicle2({
      ...vehicle2,
      [name]: name === 'name' ? value : parseFloat(value),
    });
  };

  // Handle fuel type change for vehicle 1
  const handleVehicle1FuelTypeChange = (e) => {
    const fuelType = e.target.value;
    let updatedVehicle = {
      ...vehicle1,
      fuelType,
    };
    
    // Update default values based on fuel type
    if (fuelType === 'gas') {
      updatedVehicle = {
        ...updatedVehicle,
        fuelEfficiency: 20, // mpg
        fuelCost: 4.22, // $ per gallon
        annualMaintenance: 800,
        batteryReplacementCost: 0,
        batteryReplacementYear: 0,
      };
    } else if (fuelType === 'electric') {
      updatedVehicle = {
        ...updatedVehicle,
        fuelEfficiency: 2.45, // miles per kWh
        fuelCost: 0.1343, // $ per kWh
        annualMaintenance: 400,
        batteryReplacementCost: 12000,
        batteryReplacementYear: 10,
      };
    } else if (fuelType === 'hybrid') {
      updatedVehicle = {
        ...updatedVehicle,
        fuelEfficiency: 40, // mpg
        fuelCost: 4.22, // $ per gallon
        annualMaintenance: 600,
        batteryReplacementCost: 5000,
        batteryReplacementYear: 12,
      };
    }
    
    setVehicle1(updatedVehicle);
  };

  // Handle fuel type change for vehicle 2
  const handleVehicle2FuelTypeChange = (e) => {
    const fuelType = e.target.value;
    let updatedVehicle = {
      ...vehicle2,
      fuelType,
    };
    
    // Update default values based on fuel type
    if (fuelType === 'gas') {
      updatedVehicle = {
        ...updatedVehicle,
        fuelEfficiency: 20, // mpg
        fuelCost: 4.22, // $ per gallon
        annualMaintenance: 800,
        batteryReplacementCost: 0,
        batteryReplacementYear: 0,
      };
    } else if (fuelType === 'electric') {
      updatedVehicle = {
        ...updatedVehicle,
        fuelEfficiency: 2.45, // miles per kWh
        fuelCost: 0.1343, // $ per kWh
        annualMaintenance: 400,
        batteryReplacementCost: 12000,
        batteryReplacementYear: 10,
      };
    } else if (fuelType === 'hybrid') {
      updatedVehicle = {
        ...updatedVehicle,
        fuelEfficiency: 40, // mpg
        fuelCost: 4.22, // $ per gallon
        annualMaintenance: 600,
        batteryReplacementCost: 5000,
        batteryReplacementYear: 12,
      };
    }
    
    setVehicle2(updatedVehicle);
  };

  // Format currency
  const formatCurrency = (value) => {
    return new Intl.NumberFormat('en-US', {
      style: 'currency',
      currency: 'USD',
      minimumFractionDigits: 0,
      maximumFractionDigits: 0,
    }).format(value);
  };

  return (
    <div className="w-full bg-white p-4 rounded-lg shadow">
      <h1 className="text-2xl font-bold text-center mb-6">Vehicle Total Cost of Ownership Calculator</h1>
      
      <div className="grid grid-cols-1 lg:grid-cols-2 gap-6 mb-6">
        {/* Vehicle 1 Configuration */}
        <div className="bg-gray-50 p-4 rounded-lg border border-gray-200">
          <h2 className="text-lg font-semibold mb-4 text-blue-700">Vehicle 1 Configuration</h2>
          
          <div className="mb-4">
            <label className="block mb-1 font-medium">Name:</label>
            <input
              type="text"
              name="name"
              value={vehicle1.name}
              onChange={handleVehicle1Change}
              className="w-full p-2 border rounded"
            />
          </div>

          <div className="mb-4">
            <label className="block mb-1 font-medium">MSRP ($):</label>
            <input
              type="number"
              name="msrp"
              value={vehicle1.msrp}
              onChange={handleVehicle1Change}
              className="w-full p-2 border rounded"
              min="0"
            />
          </div>

          <div className="mb-4">
            <label className="block mb-1 font-medium">Fuel Type:</label>
            <select
              value={vehicle1.fuelType}
              onChange={handleVehicle1FuelTypeChange}
              className="w-full p-2 border rounded"
            >
              <option value="gas">Gasoline</option>
              <option value="electric">Electric</option>
              <option value="hybrid">Hybrid</option>
            </select>
          </div>

          {vehicle1.fuelType === 'gas' || vehicle1.fuelType === 'hybrid' ? (
            <>
              <div className="mb-4">
                <label className="block mb-1 font-medium">Fuel Efficiency (mpg):</label>
                <input
                  type="number"
                  name="fuelEfficiency"
                  value={vehicle1.fuelEfficiency}
                  onChange={handleVehicle1Change}
                  className="w-full p-2 border rounded"
                  min="0"
                  step="0.1"
                />
              </div>
              <div className="mb-4">
                <label className="block mb-1 font-medium">Fuel Cost ($/gallon):</label>
                <input
                  type="number"
                  name="fuelCost"
                  value={vehicle1.fuelCost}
                  onChange={handleVehicle1Change}
                  className="w-full p-2 border rounded"
                  min="0"
                  step="0.01"
                />
              </div>
            </>
          ) : (
            <>
              <div className="mb-4">
                <label className="block mb-1 font-medium">Efficiency (miles/kWh):</label>
                <input
                  type="number"
                  name="fuelEfficiency"
                  value={vehicle1.fuelEfficiency}
                  onChange={handleVehicle1Change}
                  className="w-full p-2 border rounded"
                  min="0"
                  step="0.01"
                />
              </div>
              <div className="mb-4">
                <label className="block mb-1 font-medium">Electricity Cost ($/kWh):</label>
                <input
                  type="number"
                  name="fuelCost"
                  value={vehicle1.fuelCost}
                  onChange={handleVehicle1Change}
                  className="w-full p-2 border rounded"
                  min="0"
                  step="0.0001"
                />
              </div>
            </>
          )}

          <div className="mb-4">
            <label className="block mb-1 font-medium">Annual Miles:</label>
            <input
              type="number"
              name="annualMiles"
              value={vehicle1.annualMiles}
              onChange={handleVehicle1Change}
              className="w-full p-2 border rounded"
              min="0"
            />
          </div>

          {showAdvanced && (
            <>
              <div className="mb-4">
                <label className="block mb-1 font-medium">Annual Maintenance ($):</label>
                <input
                  type="number"
                  name="annualMaintenance"
                  value={vehicle1.annualMaintenance}
                  onChange={handleVehicle1Change}
                  className="w-full p-2 border rounded"
                  min="0"
                />
              </div>
              
              <div className="mb-4">
                <label className="block mb-1 font-medium">Major Maintenance Cost ($):</label>
                <input
                  type="number"
                  name="majorMaintenanceCost"
                  value={vehicle1.majorMaintenanceCost}
                  onChange={handleVehicle1Change}
                  className="w-full p-2 border rounded"
                  min="0"
                />
              </div>
              
              {(vehicle1.fuelType === 'electric' || vehicle1.fuelType === 'hybrid') && (
                <>
                  <div className="mb-4">
                    <label className="block mb-1 font-medium">Battery Replacement Year:</label>
                    <input
                      type="number"
                      name="batteryReplacementYear"
                      value={vehicle1.batteryReplacementYear}
                      onChange={handleVehicle1Change}
                      className="w-full p-2 border rounded"
                      min="0"
                    />
                  </div>
                  
                  <div className="mb-4">
                    <label className="block mb-1 font-medium">Battery Replacement Cost ($):</label>
                    <input
                      type="number"
                      name="batteryReplacementCost"
                      value={vehicle1.batteryReplacementCost}
                      onChange={handleVehicle1Change}
                      className="w-full p-2 border rounded"
                      min="0"
                    />
                  </div>
                </>
              )}
            </>
          )}
        </div>

        {/* Vehicle 2 Configuration */}
        <div className="bg-gray-50 p-4 rounded-lg border border-gray-200">
          <h2 className="text-lg font-semibold mb-4 text-green-700">Vehicle 2 Configuration</h2>
          
          <div className="mb-4">
            <label className="block mb-1 font-medium">Name:</label>
            <input
              type="text"
              name="name"
              value={vehicle2.name}
              onChange={handleVehicle2Change}
              className="w-full p-2 border rounded"
            />
          </div>

          <div className="mb-4">
            <label className="block mb-1 font-medium">MSRP ($):</label>
            <input
              type="number"
              name="msrp"
              value={vehicle2.msrp}
              onChange={handleVehicle2Change}
              className="w-full p-2 border rounded"
              min="0"
            />
          </div>

          <div className="mb-4">
            <label className="block mb-1 font-medium">Fuel Type:</label>
            <select
              value={vehicle2.fuelType}
              onChange={handleVehicle2FuelTypeChange}
              className="w-full p-2 border rounded"
            >
              <option value="gas">Gasoline</option>
              <option value="electric">Electric</option>
              <option value="hybrid">Hybrid</option>
            </select>
          </div>

          {vehicle2.fuelType === 'gas' || vehicle2.fuelType === 'hybrid' ? (
            <>
              <div className="mb-4">
                <label className="block mb-1 font-medium">Fuel Efficiency (mpg):</label>
                <input
                  type="number"
                  name="fuelEfficiency"
                  value={vehicle2.fuelEfficiency}
                  onChange={handleVehicle2Change}
                  className="w-full p-2 border rounded"
                  min="0"
                  step="0.1"
                />
              </div>
              <div className="mb-4">
                <label className="block mb-1 font-medium">Fuel Cost ($/gallon):</label>
                <input
                  type="number"
                  name="fuelCost"
                  value={vehicle2.fuelCost}
                  onChange={handleVehicle2Change}
                  className="w-full p-2 border rounded"
                  min="0"
                  step="0.01"
                />
              </div>
            </>
          ) : (
            <>
              <div className="mb-4">
                <label className="block mb-1 font-medium">Efficiency (miles/kWh):</label>
                <input
                  type="number"
                  name="fuelEfficiency"
                  value={vehicle2.fuelEfficiency}
                  onChange={handleVehicle2Change}
                  className="w-full p-2 border rounded"
                  min="0"
                  step="0.01"
                />
              </div>
              <div className="mb-4">
                <label className="block mb-1 font-medium">Electricity Cost ($/kWh):</label>
                <input
                  type="number"
                  name="fuelCost"
                  value={vehicle2.fuelCost}
                  onChange={handleVehicle2Change}
                  className="w-full p-2 border rounded"
                  min="0"
                  step="0.0001"
                />
              </div>
            </>
          )}

          <div className="mb-4">
            <label className="block mb-1 font-medium">Annual Miles:</label>
            <input
              type="number"
              name="annualMiles"
              value={vehicle2.annualMiles}
              onChange={handleVehicle2Change}
              className="w-full p-2 border rounded"
              min="0"
            />
          </div>

          {showAdvanced && (
            <>
              <div className="mb-4">
                <label className="block mb-1 font-medium">Annual Maintenance ($):</label>
                <input
                  type="number"
                  name="annualMaintenance"
                  value={vehicle2.annualMaintenance}
                  onChange={handleVehicle2Change}
                  className="w-full p-2 border rounded"
                  min="0"
                />
              </div>
              
              <div className="mb-4">
                <label className="block mb-1 font-medium">Major Maintenance Cost ($):</label>
                <input
                  type="number"
                  name="majorMaintenanceCost"
                  value={vehicle2.majorMaintenanceCost}
                  onChange={handleVehicle2Change}
                  className="w-full p-2 border rounded"
                  min="0"
                />
              </div>
              
              {(vehicle2.fuelType === 'electric' || vehicle2.fuelType === 'hybrid') && (
                <>
                  <div className="mb-4">
                    <label className="block mb-1 font-medium">Battery Replacement Year:</label>
                    <input
                      type="number"
                      name="batteryReplacementYear"
                      value={vehicle2.batteryReplacementYear}
                      onChange={handleVehicle2Change}
                      className="w-full p-2 border rounded"
                      min="0"
                    />
                  </div>
                  
                  <div className="mb-4">
                    <label className="block mb-1 font-medium">Battery Replacement Cost ($):</label>
                    <input
                      type="number"
                      name="batteryReplacementCost"
                      value={vehicle2.batteryReplacementCost}
                      onChange={handleVehicle2Change}
                      className="w-full p-2 border rounded"
                      min="0"
                    />
                  </div>
                </>
              )}
            </>
          )}
        </div>
      </div>
      
      {/* Common Parameters */}
      <div className="bg-gray-50 p-4 rounded-lg border border-gray-200 mb-6">
        <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
          <div>
            <label className="block mb-1 font-medium">Years to Calculate:</label>
            <input
              type="number"
              value={years}
              onChange={(e) => setYears(parseInt(e.target.value))}
              className="w-full p-2 border rounded"
              min="1"
              max="30"
            />
          </div>
          
          <div>
            <label className="block mb-1 font-medium">EV Tax Credit ($):</label>
            <input
              type="number"
              value={taxCredit}
              onChange={(e) => setTaxCredit(parseFloat(e.target.value))}
              className="w-full p-2 border rounded"
              min="0"
              max="7500"
              step="500"
            />
          </div>
          
          <div className="flex items-end">
            <button
              onClick={() => setShowAdvanced(!showAdvanced)}
              className="p-2 bg-gray-200 rounded hover:bg-gray-300 w-full"
            >
              {showAdvanced ? 'Hide Advanced Options' : 'Show Advanced Options'}
            </button>
          </div>
        </div>
      </div>
      
      {/* Results Chart */}
      <div className="bg-white p-4 mb-6">
        <h2 className="text-xl font-semibold mb-4 text-center">Total Cost of Ownership Comparison</h2>
        
        <div className="h-96 mb-6">
          <ResponsiveContainer width="100%" height="100%">
            <LineChart
              data={data}
              margin={{ top: 20, right: 30, left: 20, bottom: 10 }}
            >
              <CartesianGrid strokeDasharray="3 3" />
              <XAxis 
                dataKey="year" 
                label={{ value: 'Years of Ownership', position: 'insideBottom', offset: -10 }} 
              />
              <YAxis 
                label={{ value: 'Cumulative Cost ($)', angle: -90, position: 'insideLeft' }}
                tickFormatter={(value) => `$${(value/1000).toFixed(0)}k`}
              />
              <Tooltip 
                formatter={(value) => formatCurrency(value)} 
                labelFormatter={(value) => `Year ${value}`}
              />
              <Legend verticalAlign="top" height={36} />
              <Line
                type="monotone"
                dataKey={vehicle1.name}
                stroke={vehicle1.color}
                strokeWidth={2}
                activeDot={{ r: 8 }}
              />
              <Line
                type="monotone"
                dataKey={vehicle2.name}
                stroke={vehicle2.color}
                strokeWidth={2}
                activeDot={{ r: 8 }}
              />
              {crossoverYear && (
                <ReferenceLine 
                  x={crossoverYear} 
                  stroke="red" 
                  strokeDasharray="3 3" 
                  label={{ value: 'Breakeven Point', position: 'top' }} 
                />
              )}
            </LineChart>
          </ResponsiveContainer>
        </div>
        
        <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
          {/* Vehicle 1 Summary */}
          <div className="bg-gray-50 p-4 rounded-lg border border-gray-200">
            <h3 className="font-semibold text-lg mb-3 text-blue-700">{vehicle1.name} Summary</h3>
            <div className="grid grid-cols-2 gap-2">
              <div className="font-medium">Initial Cost:</div>
              <div>{formatCurrency(vehicle1.msrp)}</div>
              
              <div className="font-medium">Annual Fuel Cost:</div>
              <div>{formatCurrency(calculateAnnualFuelCost(vehicle1))}</div>
              
              <div className="font-medium">Annual Maintenance:</div>
              <div>{formatCurrency(vehicle1.annualMaintenance)}</div>
              
              {vehicle1.batteryReplacementYear > 0 && (
                <>
                  <div className="font-medium">Battery Replacement:</div>
                  <div>Year {vehicle1.batteryReplacementYear} ({formatCurrency(vehicle1.batteryReplacementCost)})</div>
                </>
              )}
              
              <div className="font-medium">Total at Year {years}:</div>
              <div className="font-bold">{data.length > 0 ? formatCurrency(data[years][vehicle1.name]) : 'Calculating...'}</div>
            </div>
          </div>
          
          {/* Vehicle 2 Summary */}
          <div className="bg-gray-50 p-4 rounded-lg border border-gray-200">
            <h3 className="font-semibold text-lg mb-3 text-green-700">{vehicle2.name} Summary</h3>
            <div className="grid grid-cols-2 gap-2">
              <div className="font-medium">Initial Cost:</div>
              <div>
                {formatCurrency(vehicle2.msrp)}
                {(vehicle2.fuelType === 'electric' || vehicle2.fuelType === 'phev') && taxCredit > 0 && (
                  <span className="text-green-600 ml-1">(-{formatCurrency(taxCredit)} tax credit)</span>
                )}
              </div>
              
              <div className="font-medium">Annual Fuel Cost:</div>
              <div>{formatCurrency(calculateAnnualFuelCost(vehicle2))}</div>
              
              <div className="font-medium">Annual Maintenance:</div>
              <div>{formatCurrency(vehicle2.annualMaintenance)}</div>
              
              {vehicle2.batteryReplacementYear > 0 && (
                <>
                  <div className="font-medium">Battery Replacement:</div>
                  <div>Year {vehicle2.batteryReplacementYear} ({formatCurrency(vehicle2.batteryReplacementCost)})</div>
                </>
              )}
              
              <div className="font-medium">Total at Year {years}:</div>
              <div className="font-bold">{data.length > 0 ? formatCurrency(data[years][vehicle2.name]) : 'Calculating...'}</div>
            </div>
          </div>
        </div>
        
        {/* Analysis */}
        <div className="mt-6 p-4 bg-gray-50 rounded-lg border border-gray-200">
          <h3 className="font-semibold text-lg mb-3">Ownership Analysis</h3>
          
          {data.length > 0 && (
            <>
              <p className="mb-2">
                The difference in {years}-year total cost of ownership between {vehicle1.name} and {vehicle2.name} is: 
                <span className="font-bold ml-1">
                  {formatCurrency(Math.abs(data[years][vehicle1.name] - data[years][vehicle2.name]))}
                  {data[years][vehicle1.name] > data[years][vehicle2.name] ? ' in favor of ' + vehicle2.name : ' in favor of ' + vehicle1.name}
                </span>
              </p>
              
              {crossoverYear ? (
                <p className="mb-2">
                  {vehicle2.name} becomes more economical than {vehicle1.name} after approximately 
                  <span className="font-bold ml-1">
                    {crossoverYear} {crossoverYear === 1 ? 'year' : 'years'} of ownership
                  </span>.
                </p>
              ) : (
                <p className="mb-2">
                  Based on the current inputs, {data[years][vehicle1.name] < data[years][vehicle2.name] ? vehicle1.name : vehicle2.name} remains more economical throughout the entire {years}-year period.
                </p>
              )}
              
              <p className="text-sm text-gray-600 mt-4">
                Note: This analysis includes the estimated costs of fuel, maintenance, and major repairs over time.
                Real-world results may vary based on driving conditions, maintenance practices, and regional cost differences.
              </p>
            </>
          )}
        </div>
      </div>
    </div>
  );
};

export default VehicleTCOCalculator;
