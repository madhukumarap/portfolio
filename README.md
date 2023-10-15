# DOCUMENTATION OF BACKEND INTERNSHIP PROJECT
# Import required libraries at the beginning of the code
import cv2  # OpenCV for computer vision tasks
from PIL import Image, ImageEnhance  # PIL (Python Imaging Library) for image processing
import numpy as np  # NumPy for numerical operations
import unittest  # For writing and running unit tests
import os  # For working with the file system
from multiprocessing import Pool  # For parallel processing of images

# Define functions for image processing
def load_image(image_path):
    """
    Load an image from a file path using Pillow (PIL) library.

    Parameters:
        image_path (str): The file path to the image.

    Returns:
        Image: The loaded image as a PIL Image object.
    """
    image = Image.open(image_path)
    return image

def preprocess_image(image, target_size=(800, 600)):
    """
    Preprocess an image by resizing, converting to grayscale, and enhancing contrast and brightness.

    Parameters:
        image (Image): The input image as a PIL Image object.
        target_size (tuple): An optional target size for resizing the image.

    Returns:
        ndarray: The preprocessed image as a NumPy array.
    """
    image = image.resize(target_size, Image.LANCZOS)  # Resize with Lanczos resampling
    if image.mode != 'L':
        image = image.convert('L')  # Convert to grayscale

    enhancer = ImageEnhance.Contrast(image)
    image = enhancer.enhance(2.0)  # Increase contrast
    enhancer = ImageEnhance.Brightness(image)
    image = enhancer.enhance(1.5)  # Increase brightness

    # Convert to a NumPy array for further processing
    image_np = np.array(image)
    return image_np

def extract_marked_levels(image, min_area_threshold=None):
    """
    Extract marked levels or elements from an image using edge detection and contour analysis.

    Parameters:
        image (ndarray): The input image as a NumPy array.
        min_area_threshold (int): An optional minimum area threshold for filtering contours.

    Returns:
        list: A list of dictionaries, each containing information about a marked level.
    """
    if min_area_threshold is None:
        min_area_threshold = adjust_min_area_threshold(image)

    if len(image.shape) == 3:
        gray_image = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
    else:
        gray_image = image

    edges = cv2.Canny(gray_image, threshold1=50, threshold2=150)  # Apply Canny edge detection

    contours, _ = cv2.findContours(edges, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)  # Find contours

    marked_levels = []

    for contour in contours:
        if cv2.contourArea(contour) > min_area_threshold:
            x, y, w, h = cv2.boundingRect(contour)
            level_info = {
                'x': x,
                'y': y,
                'width': w,
                'height': h,
            }
            marked_levels.append(level_info)

    return marked_levels

def run_stock_chart_analysis(image_path):
    """
    Analyze a stock chart image by loading, preprocessing, and extracting marked levels.

    Parameters:
        image_path (str): The file path to the stock chart image.

    Returns:
        list: A list of dictionaries containing information about marked levels.
    """
    try:
        image = load_image(image_path)
        preprocessed_image = preprocess_image(image)
        marked_levels = extract_marked_levels(preprocessed_image)
        return marked_levels
    except FileNotFoundError:
        print(f"Error: Image not found at {image_path}")
        return []

def adjust_min_area_threshold(image):
    """
    Calculate a dynamic minimum area threshold based on the image size.

    Parameters:
        image (ndarray): The input image as a NumPy array.

    Returns:
        int: The calculated dynamic threshold value.
    """
    width, height = image.shape[1], image.shape[0]
    dynamic_threshold = width * height / 1000  # Adjust the constant (1000) as needed
    return dynamic_threshold

# Define a unit test class for stock chart analysis
class TestStockChartAnalysis(unittest.TestCase):
    def test_stock_chart_analysis(self):
        # Specify the directory containing the test images
        test_image_directory = 'test_images/'

        # List all image files in the directory
        image_files = [os.path.join(test_image_directory, filename) for filename in os.listdir(test_image_directory) if filename.endswith('.jpg') or filename.endswith('.png')]

        # Create a multiprocessing pool to process images in parallel
        num_processes = 4
        with Pool(processes=num_processes) as pool:
            marked_levels_list = pool.map(run_stock_chart_analysis, image_files)

        # Iterate through the results and perform assertions for each image
        for image_path, marked_levels in zip(image_files, marked_levels_list):
            with self.subTest(image_path=image_path):
                self.assertIsInstance(marked_levels, list)  # Check if the result is a list
                self.assertTrue(all(isinstance(level, dict) for level in marked_levels))  # Check if each element is a dictionary

if __name__ == '__main__':
    unittest.main()

BONUS POINTS:
We dynamically calculate the threshold based on the image size.
We define the rule or formula to calculate the threshold based on the image size 
We provide the error message if user did not give proper image path. If user didn’t  give the proper image path we provide the message.
We provide the multiple image processing the parallel we calculate the stock analysis.
Increase the project Robustness  and scalability by adding the base case.

