#include <stdio.h>
#include <stdlib.h>
#include <portaudio.h>

#define SAMPLE_RATE 58000
#define NUM_CHANNELS 1
#define RECORDING_DURATION_SECONDS 4.5

// Callback function for recording audio
int recordCallback(const void *inputBuffer, void *outputBuffer,
                   unsigned long framesPerBuffer,
                   const PaStreamCallbackTimeInfo *timeInfo,
                   PaStreamCallbackFlags statusFlags,
                   void *userData) {
    // Cast input buffer to a float pointer (assuming float32 data)
    const float *in = (const float *)inputBuffer;
    (void) outputBuffer; // Prevent unused variable warning

    // Process the recorded samples if needed
    // Here you can save the samples to a file, process them in real-time, etc.

    return paContinue;
}

int main() {
    PaError err;
    PaStream *stream;
    int numSamples = SAMPLE_RATE * RECORDING_DURATION_SECONDS;

    // Initialize PortAudio
    err = Pa_Initialize();
    if (err != paNoError) {
        printf("Error: PortAudio initialization failed\n");
        goto error;
    }

    // Open an audio stream
    err = Pa_OpenDefaultStream(&stream,
                               NUM_CHANNELS,
                               0, // Input channels
                               paFloat32, // Sample format (32-bit floating point)
                               SAMPLE_RATE,
                               paFramesPerBufferUnspecified,
                               recordCallback,
                               NULL); // No user data
    if (err != paNoError) {
        printf("Error: Unable to open PortAudio stream\n");
        goto error;
    }

    // Start the audio stream
    err = Pa_StartStream(stream);
    if (err != paNoError) {
        printf("Error: Unable to start PortAudio stream\n");
        goto error;
    }

    printf("Recording started. Recording for %f seconds...\n", RECORDING_DURATION_SECONDS);
    Pa_Sleep(RECORDING_DURATION_SECONDS * 1000); // Sleep for recording duration in milliseconds

    // Stop and close the audio stream
    err = Pa_StopStream(stream);
    if (err != paNoError) {
        printf("Error: Unable to stop PortAudio stream\n");
        goto error;
    }

    err = Pa_CloseStream(stream);
    if (err != paNoError) {
        printf("Error: Unable to close PortAudio stream\n");
        goto error;
    }

    // Terminate PortAudio
    Pa_Terminate();
    printf("Recording completed.\n");

    return 0;

error:
    Pa_Terminate();
    return 1;
}

