# Notes

#### Links
- [Python Neural Network Backpropagation](https://machinelearningmastery.com/implement-backpropagation-algorithm-scratch-python/)
- [Add Windows PATH using cmd](https://stackoverflow.com/questions/9546324/adding-a-directory-to-the-path-environment-variable-in-windows)
- [Resizing viewport in SFML](https://stackoverflow.com/questions/27785940/shapes-proportionally-resize-with-window-in-sfml-2-x)
- [OS Error Codes](https://mariadb.com/kb/en/operating-system-error-codes/)

#### C++ useful libraries
```cpp
#include <iostream>
#include <fcntl.h>
#include <io.h>
#include <clocale>
#include <vector>
#include <string>
#include <map>
#include <fstream>
#include <experimental/filesystem>
#include <cctype>
#include <cmath>
#include <complex>
```

#### C++ set up unicode
```cpp
#ifdef _WIN32
// (void) so that visual studio won't complain
(void)_setmode(_fileno(stdin), _O_U16TEXT);
(void)_setmode(_fileno(stdout), _O_U16TEXT);
#endif
std::setlocale(LC_ALL, "en_US.utf8");
```

#### SFML Voice Recorder
```cpp
class MicRecorder : public sf::SoundRecorder {
public:
	std::vector<sf::Int16> recorded;
	~MicRecorder() {
		stop();
	}

	bool onStart() override {
		return true;
	}

	bool onProcessSamples(const sf::Int16* samples, std::size_t sampleCount) override {
		for (uint16_t i = 0; i < sampleCount; i++) {
			recorded.push_back(samples[i]);
		}
		return true;
	}

	void onStop() override {}
};

int main() {
  sf::SoundBuffer sbuf;
	if (MicRecorder::isAvailable()) {
		MicRecorder recorder;
		std::wcout << L"Available Audio Devices: PRESS E TO RECORD\n";
		std::wcout << L"Changing audio device... SUCCESS: " << recorder.setDevice("OpenAL Soft on Microphone (Conexant SmartAudio HD)") << '\n';
		//recorder.setChannelCount(1);
		std::wcout << L"Current default device: " << sf::String(MicRecorder::getDefaultDevice()).toWideString() << '\n';
		for (uint32_t i = 0; i < MicRecorder::getAvailableDevices().size(); i++) {
			std::wcout << sf::String(MicRecorder::getAvailableDevices()[i]).toWideString() << '\n';
		}
		while (!sf::Keyboard::isKeyPressed(sf::Keyboard::E));
		if (!recorder.start()) {
			std::wcout << "Unable to start!\n";
			return 1;
		}
		std::wcout << L"Recording... PRESS Q TO STOP\n";
		while (!sf::Keyboard::isKeyPressed(sf::Keyboard::Q));
		std::wcout << L"Stopping...\n";
		sbuf.loadFromSamples(&recorder.recorded[0], recorder.recorded.size(), 1, 44100);
		recorder.stop();
		sf::Sound sound;
		sound.setBuffer(sbuf);
		sound.play();
  }
}
```

#### Cython rotate 3D
```py
# Rotate 3D
# Rotate vector 3 according to origin (0, 0, 0)
# RotX: Yaw, horizontal rotate. 0 degree facing Z+
# RotY: Pitch, vertical rotate. Straight
# RotZ: Roll, rotate 2D
cdef (double, double, double) rotateVec3((double, double, double) vec3, double sX, double cX, double sY, double cY, double sZ, double cZ):
  cdef:
    double x = vec3[0]
    double y = vec3[1]
    double z = vec3[2]
    double oldX = x
    double oldY = y
  x = x * cX + z * sX
  z = z * cX - oldX * sX
  y = y * cY + z * sY
  z = z * cY - oldY * sY
  oldX = x
  oldY = y
  x = oldX * cZ - oldY * -sZ
  y = oldX * -sZ + oldY * cZ
  return (x, y, z)
```

#### Cython rotate 2D
```py
# Rotate 2D
cdef (double, double) rotateVec2((double, double) vec2, double sZ, double cZ):
  return (vec2[0] * cZ + vec2[1] * sZ, vec2[0] * -sZ + vec2[1] * cZ)
```

#### Cython linear lerp
```py
double remap(double x, double amin, double amax, double bmin, double bmax):
  if (amax - amin) + bmin == 0:
    return bmin
  else:
    return (x - amin) * (bmax - bmin) / (amax - amin) + bmin
```

#### Cython check if 2D triangle's points are drawn clockwise
```py
# Checks if a triangle is clockwise
cdef char checkClockwise((double, double) a, (double, double) b, (double, double) c):
  return (c[1]-a[1]) * (b[0]-a[0]) < (b[1]-a[1]) * (c[0]-a[0])
```

#### Cython point 2D collision with triangle
```py
# Checks if a point is inside a triangle
cdef char checkPointInTriangle((double, double) point, (double, double) a, (double, double) b, (double, double) c):
  cdef:
    double d1 = (point[0] - b[0]) * (a[1] - b[1]) - (a[0] - b[0]) * (point[1] - b[1])
    double d2 = (point[0] - c[0]) * (b[1] - c[1]) - (b[0] - c[0]) * (point[1] - c[1])
    double d3 = (point[0] - a[0]) * (c[1] - a[1]) - (c[0] - a[0]) * (point[1] - a[1])
  return not (((d1 > 0) or (d2 > 0) or (d3 > 0)) and ((d1 < 0) or (d2 < 0) or (d3 < 0)))
```

#### Cython point 2D collision with polygon
```py
# Checks if a point is inside a polygon
cdef char checkPointInPolygon((double, double) point, vector[(double, double)] polygon):
  cdef:
    char c = 0
    unsigned int i = 0
    unsigned int j = polygon.size()-1
  while i < polygon.size():
    if (((polygon[i][1]>point[1]) != (polygon[j][1]>point[1])) and\
      (point[0] < (polygon[j][0]-polygon[i][0]) * (point[1]-polygon[i][1]) / (polygon[j][1]-polygon[i][1]) + polygon[i][0])):
      c = not c
    j = i
    i += 1
  return c
```

#### Cython get line triangle intersection in 3D space
```py
cdef double signed_tetra_volume((double, double, double) a, (double, double, double) b, (double, double, double) c, (double, double, double) d):
  return sign(dot(cross((b[0]-a[0], b[1]-a[1], b[2]-a[2]), (c[0]-a[0], c[1]-a[1], c[2]-a[2])), (d[0]-a[0], d[1]-a[1], d[2]-a[2]))/6)

# Give intersection point of a line and a triangle 3D plane
cdef ((double, double, double), char) checkLineTrianglePlaneIntersection((double, double, double) q1 , (double, double, double) q2, (double, double, double) p1, (double, double, double) p2, (double, double, double) p3):
  cdef:
    double s1 = signed_tetra_volume(q1,p1,p2,p3)
    double s2 = signed_tetra_volume(q2,p1,p2,p3)
    double s3 = signed_tetra_volume(q1,q2,p1,p2)
    double s4 = signed_tetra_volume(q1,q2,p2,p3)
    double s5 = signed_tetra_volume(q1,q2,p3,p1)
    (double, double, double) n = cross((p2[0]-p1[0], p2[1]-p1[1], p2[2]-p1[2]), (p3[0]-p1[0], p3[1]-p1[1], p3[2]-p1[2]))
    double t = dot((p1[0]-q1[0], p1[1]-q1[1], p1[2]-q1[2]),n) / dot((q2[0]-q1[0], q2[1]-q1[1], q2[2]-q1[2]),n)
  if s1 != s2 and s3 == s4 and s4 == s5:
    return (((q2[0]-q1[0])*t+q1[0], (q2[1]-q1[1])*t+q1[1], (q2[2]-q1[2])*t+q1[2]), 0)
  return ((0, 0, 0), 1) # Does not intersect
```

#### Cython check 2 2D lines intersect
```py
# Check if 2 lines intersect
cdef char checkLineIntersection(((double, double), (double, double)) l1, ((double, double), (double, double)) l2):
  return (((not checkClockwise(l1[0], l2[0], l2[1])) != (not checkClockwise(l1[1], l2[0], l2[1]))) and ((not checkClockwise(l1[0], l1[1], l2[0])) != (not checkClockwise(l1[0], l1[1], l2[1]))))
```
