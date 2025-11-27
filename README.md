# Geography-of-Fashion
// nyfw-storymap-react.jsx
// Single-file React component for a StoryMap about the Geography of Fashion (NYFW)
// - Uses Mapbox GL via react-map-gl
// - Tailwind CSS utility classes used for styling (no import required if your project already uses Tailwind)
// - Replace process.env.REACT_APP_MAPBOX_TOKEN with your Mapbox token or set MAPBOX_TOKEN variable
// - Sample GeoJSON venues included. Replace with your real dataset or load from an external JSON/GeoJSON file.

import React, { useState, useMemo } from "react";
import Map, { Marker, Popup, Source, Layer } from "react-map-gl";
import "mapbox-gl/dist/mapbox-gl.css";
import { MapPin } from "lucide-react";

const MAPBOX_TOKEN = process.env.REACT_APP_MAPBOX_TOKEN || "YOUR_MAPBOX_TOKEN_HERE";

// Sample GeoJSON with some NYFW-style venues (fictional/sample coordinates)
const venues = {
  type: "FeatureCollection",
  features: [
    {
      type: "Feature",
      properties: {
        id: 1,
        name: "Bryant Park Studios",
        borough: "Manhattan",
        year_range: "2010-2020",
        description:
          "Traditional NYFW hub next to midtown hospitality and transit — central and prestigious.",
      },
      geometry: { type: "Point", coordinates: [-73.9832, 40.7536] },
    },
    {
      type: "Feature",
      properties: {
        id: 2,
        name: "Spring Studios",
        borough: "Manhattan",
        year_range: "2012-2023",
        description: "Large converted industrial space that hosts runway shows and presentations.",
      },
      geometry: { type: "Point", coordinates: [-74.0055, 40.7276] },
    },
    {
      type: "Feature",
      properties: {
        id: 3,
        name: "Skylight Modern (Brooklyn)",
        borough: "Brooklyn",
        year_range: "2016-2024",
        description: "Example of southward expansion of alternative fashion events in Brooklyn.",
      },
      geometry: { type: "Point", coordinates: [-73.9969, 40.7069] },
    },
    {
      type: "Feature",
      properties: {
        id: 4,
        name: "Lincoln Center (Historic)",
        borough: "Manhattan",
        year_range: "1990-2010",
        description: "Old guard venue: ceremonial, institutionalized spaces tied to prestige.",
      },
      geometry: { type: "Point", coordinates: [-73.9845, 40.7722] },
    },
  ],
};

export default function NYFWStoryMap() {
  const [viewport, setViewport] = useState({
    longitude: -73.98,
    latitude: 40.74,
    zoom: 11.2,
  });
  const [selected, setSelected] = useState(null);
  const [boroughFilter, setBoroughFilter] = useState("All");
  const [yearFilter, setYearFilter] = useState("All");

  const boroughs = useMemo(() => {
    const set = new Set(venues.features.map((f) => f.properties.borough));
    return ["All", ...Array.from(set)];
  }, []);

  const yearRanges = ["All", "1990-2010", "2010-2020", "2016-2024", "2012-2023"];

  const filtered = useMemo(() => {
    return {
      type: "FeatureCollection",
      features: venues.features.filter((f) => {
        const byBorough = boroughFilter === "All" || f.properties.borough === boroughFilter;
        const byYear = yearFilter === "All" || f.properties.year_range === yearFilter;
        return byBorough && byYear;
      }),
    };
  }, [boroughFilter, yearFilter]);

  return (
    <div className="h-screen flex flex-col md:flex-row bg-gray-50">
      {/* Sidebar: narrative + controls */}
      <aside className="md:w-1/3 lg:w-1/4 p-4 bg-white shadow-lg z-10">
        <h1 className="text-2xl font-bold mb-2">📍 Geography of Fashion — NYFW StoryMap</h1>
        <p className="text-sm text-gray-700 mb-4">
          Explore the spatial distribution of NYFW events across New York City — compare the
          concentration in Manhattan and the expanding footprint toward Brooklyn. Use the filters to
          explore changes over time or borough.
        </p>

        <div className="mb-3">
          <label className="block text-xs font-medium text-gray-600">Borough</label>
          <select
            className="w-full mt-1 p-2 border rounded"
            value={boroughFilter}
            onChange={(e) => setBoroughFilter(e.target.value)}
          >
            {boroughs.map((b) => (
              <option key={b} value={b}>
                {b}
              </option>
            ))}
          </select>
        </div>

        <div className="mb-3">
          <label className="block text-xs font-medium text-gray-600">Year Range</label>
          <select
            className="w-full mt-1 p-2 border rounded"
            value={yearFilter}
            onChange={(e) => setYearFilter(e.target.value)}
          >
            {yearRanges.map((yr) => (
              <option key={yr} value={yr}>
                {yr}
              </option>
            ))}
          </select>
        </div>

        <div className="mb-4">
          <h2 className="font-semibold">Narrative highlights</h2>
          <ul className="list-disc list-inside text-sm text-gray-700 mt-2">
            <li>Centralization around Manhattan driven by transit and prestige.</li>
            <li>Southward / borough spread correlates with creative economies and rising costs.</li>
            <li>Industrial conversions (studios) enable larger runway shows outside traditional
              venues.</li>
          </ul>
        </div>

        <div className="text-xs text-gray-500">
          <strong>Data:</strong> Sample venues (replace with full GeoJSON). <br />
          <strong>Map:</strong> Mapbox, interactive popups, filters.
        </div>
      </aside>

      {/* Map area */}
      <main className="flex-1 relative">
        <Map
          initialViewState={viewport}
          mapStyle="mapbox://styles/mapbox/streets-v12"
          mapboxAccessToken={MAPBOX_TOKEN}
          onMove={(evt) => setViewport(evt.viewState)}
          style={{ width: "100%", height: "100%" }}
        >
          {/* Source + Layer approach for adding filtering later (optional) */}
          <Source id="venues" type="geojson" data={filtered} />

          {/* Markers */}
          {filtered.features.map((f) => (
            <Marker
              key={f.properties.id}
              longitude={f.geometry.coordinates[0]}
              latitude={f.geometry.coordinates[1]}
              anchor="bottom"
              onClick={(e) => {
                // Prevent map click from firing
                e.originalEvent.stopPropagation();
                setSelected(f);
              }}
            >
              <div className="cursor-pointer transform hover:scale-110 transition">
                <MapPin size={22} />
              </div>
            </Marker>
          ))}

          {/* Popup */}
          {selected ? (
            <Popup
              longitude={selected.geometry.coordinates[0]}
              latitude={selected.geometry.coordinates[1]}
              anchor="top"
              onClose={() => setSelected(null)}
              closeOnClick={false}
            >
              <div className="max-w-xs">
                <h3 className="font-bold">{selected.properties.name}</h3>
                <p className="text-sm">{selected.properties.description}</p>
                <p className="text-xs text-gray-500 mt-1">{selected.properties.borough} • {selected.properties.year_range}</p>
              </div>
            </Popup>
          ) : null}
        </Map>

        {/* Floating credits / legend */}
        <div className="absolute right-4 bottom-4 bg-white p-2 rounded shadow text-xs">
          <div className="flex items-center gap-2">
            <MapPin size={14} />
            <span>Sample venue</span>
          </div>
        </div>
      </main>
    </div>
  );
}
