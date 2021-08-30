#include <iostream>
#include <curl/curl.h>
#include <fstream>
#include <string>
#include <thread>
#include <filesystem>

std::string steamLoginSecure, apiKey, steamID64, sessionID;

void changeAvatar( std::string file )
{
	CURL* curl;
	CURLcode result;

	struct curl_httppost* formPost = NULL;
	struct curl_httppost* lastptr = NULL;

	curl_global_init( CURL_GLOBAL_ALL );

	curl = curl_easy_init();

	if( curl )
	{
		curl_easy_setopt( curl, CURLOPT_URL, "https://steamcommunity.com/actions/FileUploader" );
		curl_easy_setopt( curl, CURLOPT_COOKIE, ( "sessionid=" + sessionID + ";steamLoginSecure=" + steamLoginSecure ).c_str() );

		curl_formadd( &formPost, &lastptr, CURLFORM_COPYNAME, "MAX_FILE_SIZE", CURLFORM_COPYCONTENTS, "1048576", CURLFORM_END );
		curl_formadd( &formPost, &lastptr, CURLFORM_COPYNAME, "type", CURLFORM_COPYCONTENTS, "player_avatar_image", CURLFORM_END );
		curl_formadd( &formPost, &lastptr, CURLFORM_COPYNAME, "sId", CURLFORM_COPYCONTENTS, steamID64.c_str(), CURLFORM_END );
		curl_formadd( &formPost, &lastptr, CURLFORM_COPYNAME, "sessionid", CURLFORM_COPYCONTENTS, sessionID.c_str(), CURLFORM_END );
		curl_formadd( &formPost, &lastptr, CURLFORM_COPYNAME, "doSub", CURLFORM_COPYCONTENTS, "1", CURLFORM_END );
		curl_formadd( &formPost, &lastptr, CURLFORM_COPYNAME, "json", CURLFORM_COPYCONTENTS, "1", CURLFORM_END );
		curl_formadd( &formPost, &lastptr, CURLFORM_COPYNAME, "avatar", CURLFORM_FILE, file.c_str(), CURLFORM_END );

		curl_easy_setopt( curl, CURLOPT_HTTPPOST, formPost );

		result = curl_easy_perform( curl );

		if( result != CURLE_OK )
		{
			std::cout << curl_easy_strerror( result ) << std::endl;
		}
	
		curl_easy_cleanup( curl );
	}

	curl_global_cleanup();
}

int main( int ac, char** av )
{
	if ( not av[1] ) std::cout << "specify a file/folder"; return 1;

	std::srand( std::time( nullptr) );

	char pathBuffer[MAX_PATH];
	std::string path( pathBuffer, GetModuleFileNameA( NULL, pathBuffer, MAX_PATH ) );
	path = path.substr( 0, path.find_last_of( "\\" ) + 1 );

	std::fstream configFile( ( path + "config" ) );

	if ( not configFile ) std::cout << "config file is missing"; return 1;

	std::getline( configFile, steamLoginSecure );

	configFile.close();

	steamID64 = steamLoginSecure.substr( 0, steamLoginSecure.find_first_of( "%|" ) );

	for( int i = 1; i < 25; i++ )
	{
		sessionID += char( std::rand() % 25 ) + 97;
	}

	if( not std::filesystem::exists( av[1] ) ) std::cout << av[1] << "is not a file/folder"; return 1;

	if( std::filesystem::is_directory( av[1] ) )
	{
		std::vector<std::string> fileList;

		for ( auto& p: std::filesystem::recursive_directory_iterator( av[1] ) ) 
		{
			std::string ext = p.path().extension().string();

			if( ( ext == ".jpg" || ext == ".jpeg" || ext == ".png" || ext == ".gif" ) && ( p.file_size() * 0.001 ) <= 1024 )
			{
				fileList.push_back( p.path().string() );
			}
		}

		if( fileList.size() == 0 ) std::cout << "couldn't find any suitable files"; return 1;

		std::cout << "found "<< fileList.size() << ( fileList.size() <= 1 ? " file" : " files" ) << std::endl;

		while( true )
		{
			changeAvatar( fileList[ std::rand() % fileList.size() ] );
			std::cout << std::endl;
			std::this_thread::sleep_for( std::chrono::seconds( 60 ) );
		}
	}
	else
	{
		changeAvatar( av[1] );
	}

	return 0;
}
