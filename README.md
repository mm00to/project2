#include <iostream>
#include <string>
#include <algorithm>
#include <cmath>
#include "Image_Class.h"

using namespace std;

// 1. Grayscale Filter 
void applyGrayscaleFilter(Image& img) {
    for (int i = 0; i < img.width; ++i) {
        for (int p = 0; p < img.height; ++p) {
            unsigned int pixelAvr = (img(i, p, 0) + img(i, p, 1) + img(i, p, 2)) / 3;
            img(i, p, 0) = pixelAvr;
            img(i, p, 1) = pixelAvr;
            img(i, p, 2) = pixelAvr;
        }
    }
}

// 2. Black and White Filter 
void applyWhiteAndBlackfilter(Image& img) {
    for (int i = 0; i < img.width; ++i) {
        for (int p = 0; p < img.height; ++p) {
            unsigned int pixelAvr = (img(i, p, 0) + img(i, p, 1) + img(i, p, 2)) / 3;

            if (pixelAvr >= 127) {
                img(i, p, 0) = 255;
                img(i, p, 1) = 255;
                img(i, p, 2) = 255;
            }
            else {
                img(i, p, 0) = 0;
                img(i, p, 1) = 0;
                img(i, p, 2) = 0;
            }
        }
    }
}

// 3. Invert Color Filter 
void applyInvertFilter(Image& img) {
    for (int i = 0; i < img.width; ++i) {
        for (int p = 0; p < img.height; ++p) {
            for (int c = 0; c < 3; ++c) {
                img(i, p, c) = 255 - img(i, p, c);
            }
        }
    }
}

// 4. Frame Filter 
void applyFrameFilter(Image& img) {
    int frameThickness = min(img.width, img.height) / 20;
    if (frameThickness < 5) frameThickness = 5;

    for (int i = 0; i < img.width; ++i) {
        for (int p = 0; p < img.height; ++p) {
            if (i < frameThickness || i >= img.width - frameThickness ||
                p < frameThickness || p >= img.height - frameThickness) {
                img(i, p, 0) = 0;   
                img(i, p, 1) = 0;   
                img(i, p, 2) = 150; 
            }
        }
    }
}

// 5. Flip Filter 
void applyFlipFilter(Image& img, int type) {
    if (type == 1) { // Horizontal Flip
        for (int p = 0; p < img.height; ++p) {
            for (int i = 0; i < img.width / 2; ++i) {
                for (int c = 0; c < 3; ++c) {
                    swap(img(i, p, c), img(img.width - 1 - i, p, c));
                }
            }
        }
    }
    else if (type == 2) { // Vertical Flip
        for (int i = 0; i < img.width; ++i) {
            for (int p = 0; p < img.height / 2; ++p) {
                for (int c = 0; c < 3; ++c) {
                    swap(img(i, p, c), img(i, img.height - 1 - p, c));
                }
            }
        }
    }
}

// 6. Rotate Filter Helper Functions 
Image rotate90(const Image& oldimg) {
    Image Newimg(oldimg.height, oldimg.width);

    for (int x = 0; x < oldimg.width; ++x) {
        for (int y = 0; y < oldimg.height; ++y) {
            for (int c = 0; c < 3; ++c) {
                Newimg(oldimg.height - 1 - y, x, c) = oldimg(x, y, c);
            }
        }
    }
    return Newimg;
}

Image applyRotate(const Image& img, int angle) {
    if (angle == 90) {
        return rotate90(img);
    }
    else if (angle == 180) {
        return rotate90(rotate90(img));
    }
    else if (angle == 270) {
        return rotate90(rotate90(rotate90(img)));
    }
    return img;
}

// 7. Darken and Lighten Filter 
void applyBrightnessFilter(Image& img, int mode) {
    for (int i = 0; i < img.width; ++i) {
        for (int p = 0; p < img.height; ++p) {
            for (int c = 0; c < 3; ++c) {
                if (mode == 1) { // Lighten
                    int val = img(i, p, c) + (img(i, p, c) / 2);
                    img(i, p, c) = min(255, val);
                }
                else if (mode == 2) { // Darken
                    img(i, p, c) = img(i, p, c) / 2;
                }
            }
        }
    }
}

// 8. Resize Filter 
Image applyResizeFilter(const Image& img, int newWidth, int newHeight) {
    Image resizedImg(newWidth, newHeight);

    double scaleX = (double)img.width / newWidth;
    double scaleY = (double)img.height / newHeight;

    for (int i = 0; i < newWidth; ++i) {
        for (int p = 0; p < newHeight; ++p) {
            int oldX = (int)(i * scaleX);
            int oldY = (int)(p * scaleY);

            for (int c = 0; c < 3; ++c) {
                resizedImg(i, p, c) = img(oldX, oldY, c);
            }
        }
    }
    return resizedImg;
}

// 9. Merge Filter
void applyMergeFilter(Image& img) {
    string filename2;
    cout << "Enter the filename of the second image to merge: ";
    cin >> filename2;
    try {
        Image img2(filename2);
        int minW = min(img.width, img2.width);
        int minH = min(img.height, img2.height);
        Image mergedImg(minW, minH);
        for (int i = 0; i < minW; ++i) {
            for (int p = 0; p < minH; ++p) {
                for (int c = 0; c < 3; ++c) {
                    mergedImg(i, p, c) = (img(i, p, c) + img2(i, p, c)) / 2;
                }
            }
        }
        img = mergedImg;
        cout << "Images merged successfully!\n";
    } catch (...) {
        cout << "Error loading second image for merging.\n";
    }
}

// 10. Edge Detection Filter
void applyEdgeDetectionFilter(Image &img) {
    int Kx[3][3]={
        {-1, 0, 1},
        {-2, 0, 2},
        {-1, 0, 1}
    };
    int Ky[3][3]={
        {-1,-2,-1},
        { 0, 0, 0},
        { 1, 2, 1}
    };
    Image temporary = img;
    for(int y=1; y<img.height-1; ++y){
        for(int x=1; x<img.width-1; ++x){
            float sumX = 0;
            float sumY = 0;
            for(int Gy=-1; Gy<=1; ++Gy){
                for(int Gx=-1; Gx<=1; ++Gx){
                    int pixelX = x + Gx;
                    int pixelY = y + Gy;
                    unsigned char r = temporary.getPixel(pixelX, pixelY, 0);
                    unsigned char g = temporary.getPixel(pixelX, pixelY, 1);
                    unsigned char b = temporary.getPixel(pixelX, pixelY, 2);
                    float gray = (0.299f * r + 0.587f * g + 0.114f * b);
                    sumX += gray * Kx[Gy+1][Gx+1];
                    sumY += gray * Ky[Gy+1][Gx+1];
                }
            }
            int magnitude = static_cast<int>(sqrt(sumX * sumX + sumY * sumY));
            magnitude = std::max(0, std::min(255, magnitude));
            img.setPixel(x, y, 0, static_cast<unsigned char>(magnitude));
            img.setPixel(x, y, 1, static_cast<unsigned char>(magnitude));
            img.setPixel(x, y, 2, static_cast<unsigned char>(magnitude));
        }
    }
}

// 11. Crop Filter
void applyCropFilter(Image& img) {
    int x, y, w, h;
    cout << "Current dimensions: " << img.width << "x" << img.height << endl;
    cout << "Enter start X coordinate: ";
    cin >> x;
    cout << "Enter start Y coordinate: ";
    cin >> y;
    cout << "Enter width to crop: ";
    cin >> w;
    cout << "Enter height to crop: ";
    cin >> h;

    if (x < 0 || y < 0 || x + w > img.width || y + h > img.height || w <= 0 || h <= 0) {
        cout << "Invalid crop dimensions!\n";
        return;
    }

    Image cropped(w, h);
    for (int i = 0; i < w; ++i) {
        for (int p = 0; p < h; ++p) {
            for (int c = 0; c < 3; ++c) {
                cropped(i, p, c) = img(x + i, y + p, c);
            }
        }
    }
    img = cropped;
    cout << "Image cropped successfully!\n";
}

// 12. Blur Filter
void applyBlurFilter(Image& img) {
    Image temp = img;
    int radius = 2;
    for (int i = 0; i < img.width; ++i) {
        for (int p = 0; p < img.height; ++p) {
            int sumR = 0, sumG = 0, sumB = 0, count = 0;
            for (int dx = -radius; dx <= radius; ++dx) {
                for (int dy = -radius; dy <= radius; ++dy) {
                    int nx = i + dx;
                    int ny = p + dy;
                    if (nx >= 0 && nx < img.width && ny >= 0 && ny < img.height) {
                        sumR += temp(nx, ny, 0);
                        sumG += temp(nx, ny, 1);
                        sumB += temp(nx, ny, 2);
                        count++;
                    }
                }
            }
            img(i, p, 0) = sumR / count;
            img(i, p, 1) = sumG / count;
            img(i, p, 2) = sumB / count;
        }
    }
    cout << "Blur filter applied successfully!\n";
}

// 13. Wano Sunlight Filter
void applyWanoSunlightFilter(Image& img) {
    for (int i = 0; i < img.width; ++i) {
        for (int p = 0; p < img.height; ++p) {
            int r = img(i, p, 0) + 35;
            int g = img(i, p, 1) + 20;
            int b = img(i, p, 2) - 15;
            img(i, p, 0) = min(255, max(0, r));
            img(i, p, 1) = min(255, max(0, g));
            img(i, p, 2) = min(255, max(0, b));
        }
    }
    cout << "Wano Sunlight filter applied successfully!\n";
}

// 14. Old TV Filter
void applyOldTvFilter(Image &img){
    for(int y = 0; y < img.height; ++y){
        for(int x = 0; x < img.width; ++x){
            float fac = (y % 2 == 0) ? 0.5f : 1.0f;
            int noise = (rand() % 31) - 15;

            for(int w = 0; w < 3; ++w){
                int pixelValue = img.getPixel(x, y, w);
                int newValue = static_cast<int>(pixelValue * fac) + noise;
                newValue = std::max(0, std::min(255, newValue));
                img.setPixel(x, y, w, static_cast<unsigned char>(newValue));
            }
        }
    }
}

// 15. Purple Filter
void applyPurpleFilter(Image& img) {
    for (int i = 0; i < img.width; ++i) {
        for (int p = 0; p < img.height; ++p) {
            int r = img(i, p, 0) + 40;
            int g = static_cast<int>(img(i, p, 1) * 0.7);
            int b = img(i, p, 2) + 40;
            img(i, p, 0) = min(255, max(0, r));
            img(i, p, 1) = min(255, max(0, g));
            img(i, p, 2) = min(255, max(0, b));
        }
    }
    cout << "Purple filter applied successfully!\n";
}

// 16. Samurai Filter 
void applySamuraiFilter(Image& img) {
    for (int i = 0; i < img.width; ++i) {
        for (int p = 0; p < img.height; ++p) {
            int r = static_cast<int>(img(i, p, 0) * 0.5);
            int g = static_cast<int>(img(i, p, 1) * 0.5);
            int b = static_cast<int>(img(i, p, 2) * 0.5);
            img(i, p, 0) = min(255, max(0, r));
            img(i, p, 1) = min(255, max(0, g));
            img(i, p, 2) = min(255, max(0, b));
        }
    }
    cout << "Samurai filter applied successfully!\n";
}

// 17. Skew Filter
void applySkewFilter(Image& img) {
    double shearFactor = 0.2;
    int newWidth = img.width + (int)(img.height * shearFactor);
    Image skewed(newWidth, img.height);
    for (int i = 0; i < newWidth; ++i) {
        for (int p = 0; p < img.height; ++p) {
            skewed(i, p, 0) = 255;
            skewed(i, p, 1) = 255;
            skewed(i, p, 2) = 255;
        }
    }

    for (int y = 0; y < img.height; ++y) {
        int shift = (int)((img.height - y) * shearFactor);
        for (int x = 0; x < img.width; ++x) {
            int nx = x + shift;
            if (nx >= 0 && nx < newWidth) {
                for (int c = 0; c < 3; ++c) {
                    skewed(nx, y, c) = img(x, y, c);
                }
            }
        }
    }
    img = skewed;
    cout << "Skew filter applied successfully!\n";
}

// 18. Oil Painting Filter
void applyOilPaintingFilter(Image& img) {
    Image temp = img;
    int radius = 3;
    int levels = 8;
    for (int i = 0; i < img.width; ++i) {
        for (int p = 0; p < img.height; ++p) {
            int intensityCount[8] = {0};
            int sumR[8] = {0}, sumG[8] = {0}, sumB[8] = {0};

            for (int dx = -radius; dx <= radius; ++dx) {
                for (int dy = -radius; dy <= radius; ++dy) {
                    int nx = i + dx;
                    int ny = p + dy;
                    if (nx >= 0 && nx < img.width && ny >= 0 && ny < img.height) {
                        int r = temp(nx, ny, 0);
                        int g = temp(nx, ny, 1);
                        int b = temp(nx, ny, 2);
                        int curIntensity = ((r + g + b) / 3 * levels) / 255;
                        if (curIntensity >= levels) curIntensity = levels - 1;
                        
                        intensityCount[curIntensity]++;
                        sumR[curIntensity] += r;
                        sumG[curIntensity] += g;
                        sumB[curIntensity] += b;
                    }
                }
            }

            int maxIndex = 0;
            int maxCount = 0;
            for (int l = 0; l < levels; ++l) {
                if (intensityCount[l] > maxCount) {
                    maxCount = intensityCount[l];
                    maxIndex = l;
                }
            }

            if (maxCount > 0) {
                img(i, p, 0) = sumR[maxIndex] / maxCount;
                img(i, p, 1) = sumG[maxIndex] / maxCount;
                img(i, p, 2) = sumB[maxIndex] / maxCount;
            }
        }
    }
    cout << "Oil Painting filter applied successfully!\n";
}

bool loadImage(Image& img, Image& originalImg) {
    string filename;
    cout << "Please enter the filename of the image to load: ";
    cin >> filename;
    try {
        img = Image(filename);
        originalImg = img; // حفظ النسخة الأصلية للاستعادة مستقبلاً
        cout << "Image loaded successfully!\n";
        return true;
    }
    catch (const invalid_argument& e) {
        cout << "Error loading image. Please try again.\n\n";
        return false;
    }
}

void saveImage(Image& img) {
    string newfilename;
    cout << "Enter the new filename to save the image: ";
    cin >> newfilename;

    try {
        img.saveImage(newfilename);
        cout << "Image saved successfully as " << newfilename << endl;
    }
    catch (const exception& e) {
        cout << "Failed to save the image. Error occurred.\n";
    }
}

int main() {
    Image currentImg;
    Image originalImg; // لتخزين الصورة الأصلية
    bool AreImageloaded = false;

    cout << "***************----------------------------*******************" << endl;
    cout << "Welcome to the Image Processing Program!" << endl;
    cout << "***************----------------------------*******************" << endl;

    while (!AreImageloaded) {
        AreImageloaded = loadImage(currentImg, originalImg);
    }

    while (true) {
        cout << "\n================ Menu Options ================" << endl;
        cout << "1- Apply Grayscale filter" << endl;
        cout << "2- Apply White and Black filter" << endl;
        cout << "3- Apply Invert filter" << endl;
        cout << "4- Apply Frame filter" << endl;
        cout << "5- Apply Flip filter" << endl;
        cout << "6- Rotate Image" << endl;
        cout << "7- Apply Darken/Lighten filter" << endl;
        cout << "8- Apply Resize filter" << endl;
        cout << "9- Apply Merge filter" << endl;
        cout << "10- Apply Edge Detection filter" << endl;
        cout << "11- Apply Crop filter" << endl;
        cout << "12- Apply Blur filter" << endl;
        cout << "13- Apply Wano Sunlight filter" << endl;
        cout << "14- Apply Old TV filter" << endl;
        cout << "15- Apply Purple filter" << endl;
        cout << "16- Apply Samurai filter" << endl;
        cout << "17- Apply Skew filter" << endl;
        cout << "18- Apply Oil Painting filter" << endl;
        cout << "19- Restore Original Image (إعادة الصورة لطبيعتها)" << endl;
        cout << "20- Load / Change Image" << endl;
        cout << "21- Save / Download Image" << endl;
        cout << "22- Exit" << endl;
        cout << "Please enter your choice (1-22): ";

        int choice;
        cin >> choice;

        if (cin.fail()) {
            cin.clear();
            cin.ignore(10000, '\n');
            cout << "Invalid input. Please enter a number.\n";
            continue;
        }

        switch (choice) {
        case 1:
            applyGrayscaleFilter(currentImg);
            cout << "Grayscale filter applied successfully!\n";
            break;

        case 2:
            applyWhiteAndBlackfilter(currentImg);
            cout << "Black and white filter applied successfully!\n";
            break;

        case 3:
            applyInvertFilter(currentImg);
            cout << "Invert filter applied successfully!\n";
            break;

        case 4:
            applyFrameFilter(currentImg);
            cout << "Frame filter applied successfully!\n";
            break;

        case 5: {
            int flipType;
            cout << "Choose Flip Direction (1: Horizontal, 2: Vertical): ";
            cin >> flipType;
            if (flipType == 1 || flipType == 2) {
                applyFlipFilter(currentImg, flipType);
                cout << "Flip filter applied successfully!\n";
            }
            else {
                cout << "Invalid flip type!\n";
            }
            break;
        }

        case 6: {
            int angle;
            cout << "Enter angle to rotate img (90, 180, 270): ";
            cin >> angle;
            if (angle == 90 || angle == 180 || angle == 270) {
                currentImg = applyRotate(currentImg, angle);
                cout << "Image rotated successfully!\n";
            }
            else {
                cout << "Invalid angle!\n";
            }
            break;
        }

        case 7: {
            int mode;
            cout << "Choose Mode (1: Lighten, 2: Darken): ";
            cin >> mode;
            if (mode == 1 || mode == 2) {
                applyBrightnessFilter(currentImg, mode);
                cout << "Brightness filter applied successfully!\n";
            }
            else {
                cout << "Invalid mode!\n";
            }
            break;
        }

        case 8: {
            int newW, newH;
            cout << "Current dimensions: " << currentImg.width << "x" << currentImg.height << endl;
            cout << "Enter new Width: ";
            cin >> newW;
            cout << "Enter new Height: ";
            cin >> newH;
            if (newW > 0 && newH > 0) {
                currentImg = applyResizeFilter(currentImg, newW, newH);
                cout << "Image resized successfully to " << newW << "x" << newH << "!\n";
            }
            else {
                cout << "Dimensions must be greater than 0!\n";
            }
            break;
        }

        case 9:
            applyMergeFilter(currentImg);
            break;

        case 10:
            applyEdgeDetectionFilter(currentImg);
            cout << "Edge Detection filter applied successfully!\n";
            break;

        case 11:
            applyCropFilter(currentImg);
            break;

        case 12:
            applyBlurFilter(currentImg);
            break;

        case 13:
            applyWanoSunlightFilter(currentImg);
            break;

        case 14:
            applyOldTvFilter(currentImg);
            cout << "Old TV filter applied successfully!\n";
            break;

        case 15:
            applyPurpleFilter(currentImg);
            break;

        case 16:
            applySamuraiFilter(currentImg);
            break;

        case 17:
            applySkewFilter(currentImg);
            break;

        case 18:
            applyOilPaintingFilter(currentImg);
            break;

        case 19:
            currentImg = originalImg; // استعادة الصورة الأصلية
            cout << "Image restored to original successfully!\n";
            break;

        case 20:
            loadImage(currentImg, originalImg);
            break;

        case 21:
            saveImage(currentImg);
            break;

        case 22:
            cout << "Exiting the program. Bye!" << endl;
            return 0;

        default:
            cout << "Invalid choice. Please try again.\n";
            break;
        }
    }

    return 0;
}
