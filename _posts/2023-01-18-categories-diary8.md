---
title: "2023.01.18"
excerpt: "Hands-on 머신러닝 공부, Dacon 학습"

categories:
  - 일기
tags:
  - [일상]

permalink: /categories7/diary8/

toc: true
toc_sticky: true

date: 2023-01-18
last_modified_at: 2023-01-18
---

## [2023-01-18]

Colab Pro를 결제해서 사양이 좋은 GPU를 사용하기로 마음먹었다.

확실히 결제해서 더 좋은 GPU를 사용하는 것이 학습 시간을 줄이는거에 독보적인 역할을 하는 것 같다. 

효과가 좋은 CNN 모델을 찾는 도중, ResNet과 VGGNet 등 여러 층을 보았지만 pytorch에서 pretrained 된 efficientNet을 사용하는 것이 좋을 것 같다. 

하지만 오늘 GPU 사용량을 초과해버리는 바람에 아쉽게도 학습을 못시킬 것 같다. 

내일 당장 EfficientNet을 다운받아 사용해볼 예정이다. 

참고로 어제 회의를 했는데 mediapipe 라이브러리에서 손동작을 추출한 다음 학습을 시키는 것이 결과에 좋은 영향을 주는 것을 확인하였다. 

```python
class CustomDataset(Dataset):
    def __init__(self, video_path_list, label_list):
        self.video_path_list = video_path_list
        self.label_list = label_list
        
    def __getitem__(self, index):
        frames = self.get_video(self.video_path_list[index])
        
        if self.label_list is not None:
            label = self.label_list[index]
            return frames, label
        else:
            return frames
        
    def __len__(self):
        return len(self.video_path_list)
    
    def get_video(self, path):
        frames = []
        train_images = []
        cap = cv2.VideoCapture(path)
        for _ in range(CFG['FPS']):
            _, img = cap.read()

            # img = cv2.resize(img, (CFG['IMG_SIZE'], CFG['IMG_SIZE']))
            # img = img / 255.
            frames.append(img)
        mp_drawing = mp.solutions.drawing_utils
        mp_drawing_styles = mp.solutions.drawing_styles
        mp_hands = mp.solutions.hands
        with mp_hands.Hands(
            static_image_mode=True,
            max_num_hands=2,
            min_detection_confidence=0.5) as hands:
          for idx, file in enumerate(frames):
            # Read an image, flip it around y-axis for correct handedness output (see
            # above).
            image = cv2.flip(file, 1)
            
            # Convert the BGR image to RGB before processing.
            results = hands.process(cv2.cvtColor(image, cv2.COLOR_BGR2RGB))

            image_height, image_width, _ = image.shape
            if image_height == 120:
              black = np.zeros([120,160,3],dtype=np.float64)
            else :
              black = np.zeros([360,360,3],dtype=np.float64)
            
            if not results.multi_hand_landmarks:
              black = cv2.resize(black, (CFG['IMG_SIZE'], CFG['IMG_SIZE']))
              black = black / 255.
              train_images.append(black)
              continue
            
            annotated_image = image.copy()
            for hand_landmarks in results.multi_hand_landmarks:
              
              mp_drawing.draw_landmarks(
                    black,
                    hand_landmarks,
                    mp_hands.HAND_CONNECTIONS,
                    mp_drawing_styles.get_default_hand_landmarks_style(),
                    mp_drawing_styles.get_default_hand_connections_style())
            black = cv2.flip(black, 1)
            black = cv2.resize(black, (CFG['IMG_SIZE'], CFG['IMG_SIZE']))
            black = black / 255.
            train_images.append(black)
        return torch.FloatTensor(np.array(train_images)).permute(3, 0, 1, 2)

```

손동작 추출과 dataset 클래스를 합친 결과 코드가 위 코드이다. 