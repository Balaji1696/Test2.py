#
from django.http import JsonResponse
from .models import Destination

def get_destinations(request, account_id):
    destinations = Destination.objects.filter(account__account_id=account_id)
    destination_data = [{'url': dest.url, 'http_method': dest.http_method, 'headers': dest.headers} for dest in destinations]
    return JsonResponse({'destinations': destination_data})


from django.urls import path
from .views import get_destinations

urlpatterns = [
    path('destinations/<str:account_id>/', get_destinations),
]
from django.http import JsonResponse
import json

@csrf_exempt
def incoming_data(request):
    if request.method == 'POST':
        try:
            data = json.loads(request.body)
            secret_token = data.get('app_secret_token')
            if not secret_token:
                return JsonResponse({'error': 'Unauthenticated'}, status=401)
            account = Account.objects.get(app_secret_token=secret_token)
            if request.method == 'GET' and not isinstance(data, dict):
                return JsonResponse({'error': 'Invalid Data'}, status=400)
            destinations = Destination.objects.filter(account=account)
            for destination in destinations:
                headers = destination.headers
                response = requests.request(
                    method=destination.http_method,
                    url=destination.url,
                    headers=headers,
                    json=data
                )
        return JsonResponse({'message': 'Data sent to destinations successfully.'})
        except Exception as e:
        return JsonResponse({'error': str(e)}, status=500)

else:
return JsonResponse({'error': 'Only POST requests are allowed.'}, status=405)
urlpatterns = [
    path('server/incoming_data/', incoming_data),
]
