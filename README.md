let video;
let drops = [];
let dropdown;
let acidRainData;
let selectedCountry = '';

function setup() {
  createCanvas(640, 480);
  video = createCapture(VIDEO);
  video.size(640, 480);
  video.hide();
  
  // Create dropdown menu
  dropdown = createSelect();
  dropdown.position(10, 10);
  dropdown.option('Select Country');
  dropdown.changed(updateCountry);
  
  // Load JSON data
  acidRainData = loadJSON('acidrain.json', () => {
    // Populate dropdown with countries from JSON
    let countries = Object.keys(acidRainData);
    countries.forEach(country => {
      dropdown.option(country);
    });
  });

  // Detect user's location and get country
  if (navigator.geolocation) {
    navigator.geolocation.getCurrentPosition(getCountryByLocation);
  } else {
    console.log("Geolocation is not supported by this browser.");
  }
  
  for (let i = 0; i < 500; i++) {
    drops[i] = new Drop();
  }
}

function draw() {
  background(255);
  image(video, 0, 0, width, height);
  
  // Display the selected country
  if (selectedCountry) {
    fill(255);
    textSize(24);
    textAlign(LEFT, TOP);
    text(`Country: ${selectedCountry}`, 10, 50);
  }
  
  for (let i = 0; i < drops.length; i++) {
    drops[i].fall();
    drops[i].show();
  }
}

function updateCountry() {
  selectedCountry = dropdown.value();
  // Update drop color based on the selected country's rank
  for (let i = 0; i < drops.length; i++) {
    drops[i].updateColor();
  }
}

function getCountryByLocation(position) {
  let lat = position.coords.latitude;
  let lon = position.coords.longitude;
  
  // Use a reverse geocoding service to get the country
  let url = `https://nominatim.openstreetmap.org/reverse?lat=${lat}&lon=${lon}&format=json&addressdetails=1`;
  
  loadJSON(url, (data) => {
    if (data && data.address && data.address.country) {
      selectedCountry = data.address.country;
      // Update dropdown to show detected country
      dropdown.value(selectedCountry);
      // Update drop color based on the detected country's rank
      for (let i = 0; i < drops.length; i++) {
        drops[i].updateColor();
      }
    }
  });
}

class Drop {
  constructor() {
    this.x = random(width);
    this.y = random(-500, -50);
    this.z = random(0, 20);
    this.len = map(this.z, 0, 20, 10, 20);
    this.yspeed = map(this.z, 0, 20, 1, 20);
    this.color = color(0, 173, 255); // Default light blue color
  }

  fall() {
    this.y = this.y + this.yspeed;
    let grav = map(this.z, 0, 20, 0, 0.2);
    this.yspeed = this.yspeed + grav;

    if (this.y > height) {
      this.y = random(-200, -100);
      this.yspeed = map(this.z, 0, 20, 4, 10);
    }
  }

  show() {
    let thick = map(this.z, 0, 20, 1, 3);
    strokeWeight(thick);
    stroke(this.color);
    line(this.x, this.y, this.x, this.y + this.len);

    // Add motion blur effect
    strokeWeight(thick + 1);
    stroke(0, 50); // Semi-transparent black for blurring effect
    line(this.x, this.y + this.len * 0.5, this.x, this.y + this.len);
  }

  updateColor() {
    if (selectedCountry && acidRainData[selectedCountry]) {
      let rank = acidRainData[selectedCountry].rank;
      // Map rank to a color from light blue to green
      let blue = map(rank, 1, 100, 173, 0); // Light blue (173) to green (0)
      let green = map(rank, 1, 100, 216, 128); // Light blue-green (216) to darker green (128)
      this.color = color(0, green, blue);
    }
  }
}
