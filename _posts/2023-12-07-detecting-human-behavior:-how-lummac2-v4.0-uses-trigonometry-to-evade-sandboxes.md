---
layout: post
math: true
title: # Detecting Human Behavior: How LummaC2 v4.0 Uses Trigonometry to Evade Sandboxes
date: '2023-12-07 2:10:10 +0400'
categories: [Malware Analysis]
tags: [Evasion Techniques,Programming,Sandboxing,Threat Intelligence]
---
# Detecting Human Behavior: How LummaC2 v4.0 Uses Trigonometry to Evade Sandboxes

One of the latest innovations in malware is the use of trigonometry to detect human behavior, a technique that has been notably implemented in the LummaC2 v4.0 malware. This advanced tactic allows the malware to evade analysis by sandbox environments and avoid premature detonation. In this technical deep dive, we'll explore the intricacies of this method and discuss how vendors can adapt.

## Understanding the Basics of LummaC2 v4.0's Technique

LummaC2 v4.0 begins its evasion technique by monitoring mouse movements. The malware captures a series of cursor positions over a short interval. These positions are then used to form vectors, which in turn, are analyzed for the angles they make with each other. The underlying assumption is that human mouse movement is typically smooth and makes smaller angles, unlike the erratic or large angles that might be produced by automated scripts or sandbox emulations.

### The Mathematical Foundation

The core of LummaC2 v4.0's evasion lies in its ability to monitor and interpret mouse movements. By analyzing the cursor's trajectory and employing trigonometric principles, the malware makes educated guesses whether it's dealing with a human or a machine.

$$d(P, Q) = \sqrt{(x_2 - x_1)^2 + (y_2 - y_1)^2}$$

On the other hand, the dot product of two vectors $\overrightarrow{AB} \ and \ \overrightarrow{CD}$ , formed by the points ${A}$,${B}$,${C}$ and ${D}$, is calculated as

$\overrightarrow{AB} \cdot \overrightarrow{CD} = (B_x - A_x)(D_x - C_x) + (B_y - A_y)(D_y - C_y)$

To find the angle $\theta$ between the two vectors, we use the cosine similarity derived from the dot product:

$\cos(\theta) = \frac{\overrightarrow{AB} \cdot \overrightarrow{CD}}{\| \overrightarrow{AB} \| \times \| \overrightarrow{CD} \|}$

Below is a simple C proof of concept (PoC) that demonstrates the concept of using trigonometry to detect human-like mouse movement.

```c
#include <stdio.h>
#include <math.h>

typedef struct {
    int x;
    int y;
} Point;

double calculateEuclideanDistance(Point p1, Point p2) {
    return sqrt(pow(p2.x - p1.x, 2) + pow(p2.y - p1.y, 2));
}

double calculateDotProduct(Point p0, Point p1, Point p2, Point p3) {
    int x1 = p1.x - p0.x;
    int y1 = p1.y - p0.y;
    int x2 = p3.x - p2.x;
    int y2 = p3.y - p2.y;
    
    return x1 * x2 + y1 * y2;
}

double calculateAngle(Point p0, Point p1, Point p2, Point p3) {
    double dotProduct = calculateDotProduct(p0, p1, p2, p3);
    double distance1 = calculateEuclideanDistance(p0, p1);
    double distance2 = calculateEuclideanDistance(p2, p3);
    
    if (distance1 == 0 || distance2 == 0) {
        return 0.0;
    }
    
    double angle = acos(dotProduct / (distance1 * distance2));
    return angle; // angle in radians
}

int isHumanMouseMovement(Point *points, int size) {
    // Threshold in radians for human-like movement. 45 degrees in radians is approx 0.785398.
    double threshold = 0.785398; 

    for (int i = 0; i < size - 3; i++) {
        double angle = calculateAngle(points[i], points[i+1], points[i+2], points[i+3]);
        if (angle > threshold) {
            return 0; // Movement is not human-like
        }
    }
    
    return 1; // All movements are below the threshold, so it's human-like
}

int main() {
    Point mousePositions[5] = {
        {100, 100},
        {105, 105},
        {110, 110},
        {115, 108},
        {120, 105}
    };

    // Check if the mouse movement is human-like
    if (!isHumanMouseMovement(mousePositions, 5)) {
        printf("Mouse movement is not human-like! Exiting...\n");
        return 1; // Exit if the condition is not met
    }

    printf("Mouse movement seems human-like.\n");
    // Continue with the rest of the program...
    return 0;
}
```

### How Sandboxes Can Adapt

To counter this anti-sandbox technique, vendors must ensure that their emulation of human behavior is sophisticated enough to pass this trigonometric check. This means incorporating smooth, human-like cursor movements with varied but small angles. It also requires sandbox environments to simulate mouse movements that are less predictable and more nuanced, closely mimicking how a user would interact with their system.

### Implications for Cybersecurity

The use of trigonometry to detect human behavior illustrates the lengths to which malware authors will go to avoid detection. It underscores the need for continuous innovation in sandbox technology and detonation environments. As malware becomes more intelligent, so too must our methods for detecting and analyzing it.

Cybersecurity vendors need to consider this trigonometric technique as a potential indicator of a new breed of evasive malware. By understanding and adapting to these techniques, vendors can improve their detection mechanisms and better protect users from the latest threats.

### Conclusion

LummaC2 v4.0's use of trigonometry to evade detection is a testament to the malware's ingenuity. It challenges the cybersecurity community to develop more advanced analysis tools that can simulate human behavior convincingly. The ongoing battle between malware and cybersecurity defenses continues to evolve, and understanding these techniques is crucial for maintaining the upper hand.
